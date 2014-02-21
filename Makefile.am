AUTOMAKE_OPTIONS = foreign
SUBDIRS = man share bin
EXTRA_DIST = LICENSE README.md

.PHONY: release \
	release_build release_push release_clean \
	release_build_tarball release_build_homebrew release_build_arch \
	release_push_tarball release_push_homebrew release_push_arch \
	release_clean_tarball release_clean_homebrew release_clean_arch \
	release_build_deb release_push_deb release_clean_deb release_build_tag \
	release_push_tag release_clean_tag release_build_gh-pages \
	release_push_gh-pages release_clean_gh-pages

ORIGIN_URL=$(shell git config --get remote.origin.url)

edit_package = sed \
	-e 's|@PACKAGE[@]|$(PACKAGE)|g' \
	-e 's|@PACKAGE_VERSION[@]|$(PACKAGE_VERSION)|g' \
	-e 's|@DIST_ARCHIVES[@]|$(DIST_ARCHIVES)|g' \
	-e 's|@DIST_SHA[@]|$(DIST_SHA)|g'

release: release_build release_push release_clean

release_build: release_build_tarball release_build_homebrew release_build_arch \
	release_build_deb release_build_tag release_build_gh-pages

release_push: release_push_tarball release_push_homebrew release_push_arch \
	release_push_deb release_push_tag release_push_gh-pages

release_clean: release_clean_tarball release_clean_homebrew release_clean_arch \
	release_clean_deb release_clean_tag release_clean_gh-pages

###### Tarball
release_build_tarball: Makefile distcheck
	git clone --branch gh-pages $(ORIGIN_URL) gh-pages && \
		mkdir gh-pages/dist && \
		cp $(DIST_ARCHIVES) gh-pages/dist && \
		cd gh-pages &&\
			git add dist/$(DIST_ARCHIVES) &&\
			git commit -m "Release version $(PACKAGE_VERSION) tarball"

release_push_tarball:
	cd gh-pages &&\
		git push

release_clean_tarball:
	rm -rf gh-pages
	rm -rf $(DIST_ARCHIVES)

###### Homebrew
release_build_homebrew: DIST_SHA
	git clone git@github.com:thoughtbot/homebrew-formulae.git homebrew-formulae
	$(edit_package) homebrew/$(PACKAGE).rb.in > homebrew-formulae/Formula/$(PACKAGE).rb
	cd homebrew-formulae &&\
		git commit -am "$(PACKAGE): Release version $(PACKAGE_VERSION)"

release_push_homebrew:
	cd homebrew-formulae &&\
		git push

release_clean_homebrew:
	rm -rf homebrew-formulae
	rm -rf $(DIST_ARCHIVES)

###### Arch
release_build_arch: DIST_SHA
	git clone --branch master $(ORIGIN_URL) release-arch
	cd release-arch &&\
		$(edit_package) arch/PKGBUILD.in > arch/PKGBUILD &&\
		git commit -m "Release version $(PACKAGE_VERSION) for Arch" -- arch/PKGBUILD

release_push_arch:
	cd release-arch &&\
		git push

release_clean_arch:
	rm -rf release-arch
	rm -rf $(DIST_ARCHIVES)

###### Deb
release_build_deb: NEWS.md DIST_SHA distcheck
	cp -f NEWS.md debian/changelog && \
		mkdir -p deb-build && \
		cp $(DIST_ARCHIVES) deb-build/$(PACKAGE)_$(PACKAGE_VERSION).orig.tar.gz && \
		tar -C deb-build -xf deb-build/$(PACKAGE)_$(PACKAGE_VERSION).orig.tar.gz && \
		cp -R debian deb-build/$(PACKAGE)-$(PACKAGE_VERSION) && \
		cd deb-build/$(PACKAGE)-$(PACKAGE_VERSION) && \
		debuild -us -uc && \
		cd $(abs_srcdir) && \
		git clone --branch gh-pages $(ORIGIN_URL) gh-pages && \
		cp deb-build/$(PACKAGE)_$(PACKAGE_VERSION)*.deb gh-pages/debs && \
		cd gh-pages && \
		git add debs/$(PACKAGE)_$(PACKAGE_VERSION)*deb && \
		git commit -m "Release version $(PACKAGE_VERSION) for Debian" -- debs/$(PACKAGE)_$(PACKAGE_VERSION)*deb

release_push_deb:
	cd gh-pages && \
		git push

release_clean_deb:
	rm -rf gh-pages
	rm -rf deb-build
	rm -rf $(DIST_ARCHIVES)

###### Tag
release_build_tag:
	git tag -s -f -a v$(PACKAGE_VERSION) -m "Release $(PACKAGE_VERSION)"

release_push_tag:
	git push origin --tags

release_clean_tag:

###### Web page
### TODO: use the man/Makefile.am
release_build_gh-pages:
	git clone --branch gh-pages --single-branch . gh-pages && \
	for i in lsrc.1 mkrc.1 rcm.7 rcrc.5 rcup.1 rcdn.1; do \
	  mandoc -Thtml -Oman=%N.%S.html man/$$i > gh-pages/$$i.html ; \
	done && \
	cp gh-pages/rcm.7.html gh-pages/index.html && \
	git commit -am "HTML documentation for $(PACKAGE_VERSION)"

release_push_gh-pages:
	cd gh-pages && \
		git push -f $(ORIGIN_URL) gh-pages

release_clean_gh-pages:
	rm -rf gh-pages

DIST_SHA: Makefile distcheck
	$(eval DIST_SHA := $(shell shasum $(srcdir)/$(DIST_ARCHIVES) | cut -d' ' -f1))
