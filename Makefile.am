NULL =

ACLOCAL_AMFLAGS = -I m4

if ENABLE_SERVER
    IPASERVER_SUBDIRS = ipaserver
    SERVER_SUBDIRS = daemons init install
endif

if WITH_IPATESTS
    IPATESTS_SUBDIRS = ipatests
endif

IPACLIENT_SUBDIRS = ipaclient ipalib ipaplatform ipapython
PYTHON_SUBDIRS = $(IPACLIENT_SUBDIRS) $(IPATESTS_SUBDIRS) $(IPASERVER_SUBDIRS)
IPA_PLACEHOLDERS = freeipa ipa ipaserver ipatests
SUBDIRS = asn1 util client contrib po pypi $(PYTHON_SUBDIRS) $(SERVER_SUBDIRS)

GENERATED_PYTHON_FILES = \
	$(top_builddir)/ipaplatform/override.py \
	$(top_builddir)/ipapython/version.py \
	$(top_builddir)/makeaci \
	$(top_builddir)/makeapi \
	$(NULL)

MOSTLYCLEANFILES = ipasetup.pyc ipasetup.pyo \
		   pylint_plugins.pyc pylint_plugins.pyo

# user-facing scripts
nodist_bin_SCRIPTS = ipa

# files required for build but not installed
nodist_noinst_SCRIPTS = \
	makeapi \
	makeaci \
	$(NULL)

dist_noinst_SCRIPTS = \
	make-doc \
	make-test \
	pylint_plugins.py \
	$(NULL)

# templates
dist_noinst_DATA = \
	ipa.in		\
	makeaci.in	\
	makeapi.in	\
	$(NULL)

ipasetup.py: ipasetup.py.in $(CONFIG_STATUS)
	$(AM_V_GEN)sed						\
		-e 's|@VERSION[@]|$(VERSION)|g'			\
		$< > $@

.wheelconstraints: .wheelconstraints.in $(CONFIG_STATUS)
	$(AM_V_GEN)sed						\
		-e 's|@VERSION[@]|$(VERSION)|g'			\
		$< > $@

EXTRA_DIST = .mailmap \
	     ACI.txt \
	     API.txt \
	     BUILD.txt \
	     config.rpath \
	     README.md \
	     Contributors.txt \
	     COPYING.openssl \
	     contrib \
	     doc \
	     freeipa.spec.in \
	     ipasetup.py.in \
	     pylintrc \
	     .wheelconstraints.in

clean-local:
	rm -rf "$(RPMBUILD)"
	rm -rf "$(top_builddir)/dist"
	rm -rf "$(top_builddir)/.tox"
	rm -rf "$(top_srcdir)/__pycache__"
	rm -f "$(top_builddir)"/$(PACKAGE)-*.tar.gz

# convenience targets for RPM build
.PHONY: rpmroot rpmdistdir version-update _dist-version-bakein _rpms-prep \
	rpms _rpms-body srpms _srpms-body
RPMBUILD ?= $(abs_builddir)/rpmbuild
TARBALL = $(PACKAGE)-$(VERSION).tar.gz

freeipa.spec: freeipa.spec.in $(top_builddir)/$(CONFIG_STATUS)
	$(AM_V_GEN)sed						\
		-e 's|@VERSION[@]|$(VERSION)|g'			\
		-e 's|@VENDOR_SUFFIX[@]|$(VENDOR_SUFFIX)|g'	\
		$< > $@

rpmroot:
	mkdir -p $(RPMBUILD)/BUILD
	mkdir -p $(RPMBUILD)/RPMS
	mkdir -p $(RPMBUILD)/SOURCES
	mkdir -p $(RPMBUILD)/SPECS
	mkdir -p $(RPMBUILD)/SRPMS

rpmdistdir:
	mkdir -p $(top_builddir)/dist/rpms
	mkdir -p $(top_builddir)/dist/srpms

# force IPA version re-generation (useful for build from Git)
version-update:
	touch $(srcdir)/VERSION.m4

# convert Git snapshot version to static value usable from inside of tarball
_dist-version-bakein:
if !IS_GIT_SNAPSHOT
	@echo "version-bakein target requires IPA_VERSION_IS_GIT_SNAPSHOT=yes"
	exit 1
endif !IS_GIT_SNAPSHOT
	chmod u+w $(top_distdir)/VERSION.m4
	$(SED) -e 's/^define(IPA_VERSION_IS_GIT_SNAPSHOT,.*)/define(IPA_VERSION_IS_GIT_SNAPSHOT, no)/' -i $(top_distdir)/VERSION.m4
	$(SED) -e 's/^define(IPA_VERSION_PRE_RELEASE,\(.*\))/define(IPA_VERSION_PRE_RELEASE,\1.$(GIT_VERSION))/' -i $(top_distdir)/VERSION.m4
	cd $(top_distdir) && autoconf  # re-generate configure from VERSION.m4

if IS_GIT_SNAPSHOT
VERSION_UPDATE_TARGET = version-update
VERSION_BAKEIN_TARGET = _dist-version-bakein
endif IS_GIT_SNAPSHOT

# HACK to support IPA_VERSION_IS_GIT_SNAPSHOT:
# touch VERSION.m4 will reexecute configure and change $(VERSION) used by dist
# but it will not change $(VERSION) in already running target rpms.
# We need to record new $(TARBALL) value used by dist for furher use
# in rpms target.
dist-hook: $(VERSION_BAKEIN_TARGET)
	echo "$(TARBALL)" > $(top_builddir)/.tarball_name
	echo "$(VERSION)" > $(top_builddir)/.version

_rpms-prep: dist-gzip rpmroot rpmdistdir freeipa.spec
	cp $(top_builddir)/$$(cat $(top_builddir)/.tarball_name) $(RPMBUILD)/SOURCES/
	rm -f $(top_builddir)/.tarball_name

rpms: $(VERSION_UPDATE_TARGET)
	$(MAKE) _rpms-body

_rpms-body: _rpms-prep
	rpmbuild --define "_topdir $(RPMBUILD)" -ba $(top_builddir)/$(PACKAGE).spec  $(RPMBUILD_OPTS)
	cp $(RPMBUILD)/RPMS/*/*$$(cat $(top_builddir)/.version)*.rpm $(top_builddir)/dist/rpms/
	cp $(RPMBUILD)/SRPMS/*$$(cat $(top_builddir)/.version)*.src.rpm $(top_builddir)/dist/srpms/
	rm -f rm -f $(top_builddir)/.version

srpms: $(VERSION_UPDATE_TARGET)
	$(MAKE) _srpms-body

_srpms-body: _rpms-prep
	rpmbuild --define "_topdir $(RPMBUILD)" -bs $(top_builddir)/$(PACKAGE).spec $(RPMBUILD_OPTS)
	cp $(RPMBUILD)/SRPMS/*$$(cat $(top_builddir)/.version)*.src.rpm $(top_builddir)/dist/srpms/
	rm -f rm -f $(top_builddir)/.version

.PHONY: lite-server
lite-server: $(GENERATED_PYTHON_FILES)
	+$(MAKE) -C $(top_builddir)/install/ui
	PYTHONPATH=$(top_srcdir) $(PYTHON) -bb \
	    contrib/lite-server.py $(LITESERVER_ARGS)

.PHONY: lint
if WITH_POLINT
POLINT_TARGET = polint
endif WITH_POLINT
if WITH_PYLINT
PYLINT_TARGET = pylint
endif WITH_PYLINT
if WITH_JSLINT
JSLINT_TARGET = jslint
endif WITH_JSLINT
lint: acilint apilint $(POLINT_TARGET) $(PYLINT_TARGET) $(JSLINT_TARGET) rpmlint yamllint

.PHONY: devcheck
devcheck: all
if ! WITH_POLINT
	@echo "ERROR: polint not available"; exit 1
endif
if ! WITH_PYLINT
	@echo "ERROR: pylint not available"; exit 1
endif
if ! WITH_JSLINT
	@echo "ERROR: jslint not available"; exit 1
endif
	@ # just tests, aci, api and pylint on Python 3
	PYTHONPATH=$(abspath $(top_srcdir)) $(PYTHON) ipatests/ipa-run-tests \
	    --ipaclient-unittests
	$(MAKE) $(AM_MAKEFLAGS) acilint apilint polint pylint jslint rpmlint yamllint check
	@echo "All tests passed."

.PHONY: fastcheck fasttest fastlint
fastcheck:
	@$(MAKE) -j1 $(AM_MAKEFLAGS) fastlint rpmlint yamllint fasttest apilint acilint

fasttest: $(GENERATED_PYTHON_FILES) ipasetup.py
	@ # --ignore doubles speed of total test run compared to pytest.skip()
	@ # on module.
	PYTHONPATH=$(abspath $(top_srcdir)) $(PYTHON) ipatests/ipa-run-tests \
	    --skip-ipaapi \
	    --ignore $(abspath $(top_srcdir))/ipatests/test_integration \
	    --ignore $(abspath $(top_srcdir))/ipatests/test_xmlrpc

fastlint: $(GENERATED_PYTHON_FILES) ipasetup.py
if ! WITH_PYLINT
	@echo "ERROR: pylint not available"; exit 1
endif
	@echo "Fast linting with $(PYTHON) from branch '$(GIT_BRANCH)'"

	@MERGEBASE=$$(git merge-base --fork-point $(GIT_BRANCH)); \
	FILES=$$(git diff --name-only --diff-filter=d $${MERGEBASE} \
	    | grep -E '\.py$$'); \
	if [ -n "$${FILES}" ]; then \
	    echo -e "Fast linting files:\n$${FILES}\n"; \
	    echo "pycodestyle"; \
	    echo "-----------"; \
	    git diff -U0 $${MERGEBASE} | \
	        $(PYTHON) -m pycodestyle --diff || exit $$?; \
	    echo -e "\npylint"; \
	    echo "------"; \
	    PYTHONPATH=$(abspath $(top_srcdir)) $(PYTHON) -m pylint \
	        --rcfile=$(top_srcdir)/pylintrc \
	        --load-plugins pylint_plugins \
	        $${FILES} || exit $$?; \
	else \
	    echo "No modified Python files found"; \
	fi


.PHONY: $(top_builddir)/ipaplatform/override.py
$(top_builddir)/ipaplatform/override.py:
	(cd $(top_builddir)/ipaplatform && make override.py)

.PHONY: $(top_builddir)/ipapython/version.py
$(top_builddir)/ipapython/version.py:
	(cd $(top_builddir)/ipapython && make version.py)

.PHONY: acilint
acilint: $(GENERATED_PYTHON_FILES)
	cd $(srcdir); $(PYTHON) ./makeaci --validate

.PHONY: apilint
apilint: $(GENERATED_PYTHON_FILES)
	cd $(srcdir); $(PYTHON) ./makeapi --validate

.PHONY: polint
polint:
	$(MAKE) -C $(srcdir)/po PYTHON=$(PYTHON) \
	    validate-src-strings validate-po test-gettext

.PHONY: rpmlint
rpmlint: freeipa.spec
	rpmlint ./$<

YAML_FILES = \
	$(top_srcdir)/.travis.yml \
	$(wildcard $(top_srcdir)/.*.yaml) \
	$(wildcard $(top_srcdir)/ipatests/prci_definitions/*.yaml)

.PHONY: yamllint
yamllint: $(YAML_FILES)
	@for YAML in $^; do \
		$(PYTHON) -c "import yaml; f = open('$${YAML}'); yaml.safe_load(f); f.close()" || exit 1; \
	done

# Run pylint for all python files. Finds all python files/packages, skips
# folders rpmbuild, freeipa-* and dist. Skip (match, but don't print) .*,
# *.in, *~. Finally print all python files, including scripts that do not
# have python extension.

.PHONY: pylint

if WITH_PYLINT
pylint: $(GENERATED_PYTHON_FILES) ipasetup.py
	FILES=`find $(top_srcdir) \
		-type d -exec test -e '{}/__init__.py' \; -print -prune -o \
		-path './rpmbuild' -prune -o \
		-path './freeipa-*' -prune -o \
		-path './dist' -prune -o \
		-path './pypi' -prune -o \
		-path './.tox' -prune -o \
		-name '.*' -o \
		-name '*.in' -o \
		-name '*~' -o \
		-name '*.py' -print -o \
		-type f -exec grep -qsm1 '^#!.*\bpython' '{}' \; -print`; \
	echo "Pylint on $(PYTHON) is running, please wait ..."; \
	PYTHONPATH=$(top_srcdir) $(PYTHON) -m pylint \
		--rcfile=$(top_srcdir)/pylintrc \
		--load-plugins pylint_plugins \
		$${FILES}
endif  # WITH_PYLINT

.PHONY: jslint jslint-ui jslint-ui-test jslint-html \
	$(top_builddir)/install/ui/src/libs/loader.js

if WITH_JSLINT
jslint: jslint-ui jslint-ui-test jslint-html

$(top_builddir)/install/ui/src/libs/loader.js:
	(cd $(top_builddir)/install/ui/src/libs && make loader.js)

# create temporary symlinks to allow jslint to find libs/loader.js
jslint-ui: $(top_builddir)/install/ui/src/libs/loader.js
	cd $(top_srcdir)/install/ui;				\
	jsl -nologo -nosummary -nofilelisting -conf jsl.conf;

jslint-ui-test:
	cd $(top_srcdir)/install/ui/test;			\
	jsl -nologo -nosummary -nofilelisting -conf jsl.conf

jslint-html:
	cd $(top_srcdir)/install/html; 				\
	jsl -nologo -nosummary -nofilelisting -conf jsl.conf
endif  # WITH_JSLINT

.PHONY: bdist_wheel wheel_bundle wheel_placeholder pypi_packages
WHEELDISTDIR = $(top_builddir)/dist/wheels
WHEELPYPIDIR = $(top_builddir)/dist/pypi
WHEELBUNDLEDIR = $(top_builddir)/dist/bundle

@MK_IFEQ@ ($(IPA_SERVER_WHEELS),1)
    IPA_WHEEL_PACKAGES @MK_ASSIGN@ $(IPACLIENT_SUBDIRS) ipaplatform ipaserver
    IPA_OMIT_INSTALL @MK_ASSIGN@ 0
@MK_ELSE@
    IPA_WHEEL_PACKAGES @MK_ASSIGN@ $(IPACLIENT_SUBDIRS)
    IPA_OMIT_INSTALL @MK_ASSIGN@ 1
@MK_ENDIF@

# additional wheels for bundle, e.g. IPA_EXTRA_WHEELS="ipatests[webui] pylint"
IPA_EXTRA_WHEELS=

$(WHEELDISTDIR):
	mkdir -p $(WHEELDISTDIR)

$(WHEELBUNDLEDIR):
	mkdir -p $(WHEELBUNDLEDIR)

$(WHEELPYPIDIR):
	mkdir -p $(WHEELPYPIDIR)

bdist_wheel: $(WHEELDISTDIR)
	rm -f $(foreach item,$(IPA_WHEEL_PACKAGES) ipatests,$(WHEELDISTDIR)/$(item)-*.whl)
	export IPA_OMIT_INSTALL=$(IPA_OMIT_INSTALL); \
	for dir in $(IPA_WHEEL_PACKAGES) ipatests; do \
	    $(MAKE) $(AM_MAKEFLAGS) -C $${dir} $@ || exit 1; \
	done

wheel_bundle: $(WHEELBUNDLEDIR) bdist_wheel .wheelconstraints
	rm -f $(foreach item,$(IPA_WHEEL_PACKAGES) ipatests,$(WHEELBUNDLEDIR)/$(item)-*.whl)
	@# dbus-python sometimes fails when MAKEFLAGS is set to -j2 or higher
	MAKEFLAGS= $(PYTHON) -m pip wheel \
	    --disable-pip-version-check \
	    --constraint .wheelconstraints \
	    --find-links $(WHEELDISTDIR) \
	    --find-links $(WHEELBUNDLEDIR) \
	    --wheel-dir $(WHEELBUNDLEDIR) \
	    $(IPA_EXTRA_WHEELS) $(IPA_WHEEL_PACKAGES)

pypi_packages: $(WHEELPYPIDIR) .wheelconstraints
	rm -f $(WHEELPYPIDIR)/*
	for dir in $(IPACLIENT_SUBDIRS); do \
	    $(MAKE) $(AM_MAKEFLAGS) \
	        IPA_OMIT_INSTALL=1 WHEELDISTDIR="$(abspath $(WHEELPYPIDIR))" \
	        -C $${dir} bdist_wheel || exit 1; \
	done
	for dir in $(IPA_PLACEHOLDERS); do \
	    $(MAKE) $(AM_MAKEFLAGS) \
	        IPA_OMIT_INSTALL=1 WHEELDISTDIR="$(abspath $(WHEELPYPIDIR))" \
	        -C $(top_srcdir)/pypi/$${dir} bdist_wheel || exit 1; \
	done
	@echo -e "\n\nTo upload packages to PyPI, run:\n"
	@echo -e "    twine upload $(WHEELPYPIDIR)/*-$(VERSION)-py2.py3-none-any.whl\n"

.PHONY: python_install
python_install:
	for dir in $(PYTHON_SUBDIRS); do \
	    $(MAKE) $(AM_MAKEFLAGS) -C $${dir} install || exit 1; \
	done

.PHONY:
strip-po:
	$(MAKE) -C po strip-po

PYTHON_SHEBANG = 					\
	ipa	\
	makeaci	\
	makeapi	\
	$(NULL)

CLEANFILES = $(PYTHON_SHEBANG)

include $(top_srcdir)/Makefile.pythonscripts.am
