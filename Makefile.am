# Copyright (c) 2012 Yubico AB
# All rights reserved.
# 
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are
# met:
# 
#     * Redistributions of source code must retain the above copyright
#       notice, this list of conditions and the following disclaimer.
# 
#     * Redistributions in binary form must reproduce the above
#       copyright notice, this list of conditions and the following
#       disclaimer in the documentation and/or other materials provided
#       with the distribution.
# 
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

AM_CFLAGS = $(WERROR_CFLAGS) $(WARN_CFLAGS)
AM_CPPFLAGS = $(PCSC_CFLAGS)

bin_PROGRAMS = ykneo-ccid-modeswitch
ykneo_ccid_modeswitch_SOURCES = NEO_Config.c neo_if.h ykdef.h
ykneo_ccid_modeswitch_SOURCES += cmdline.ggo cmdline.c cmdline.h
ykneo_ccid_modeswitch_LDADD = $(PCSC_LIBS)

cmdline.c cmdline.h: cmdline.ggo Makefile.am
	gengetopt --input $^

BUILT_SOURCES = cmdline.c cmdline.h
MAINTAINERCLEANFILES = $(BUILT_SOURCES)

# Doc.

dist_man_MANS = ykneo-ccid-modeswitch.1
MAINTAINERCLEANFILES += $(dist_man_MANS)

ykneo-ccid-modeswitch.1: $(ykneo_ccid_modeswitch_SOURCES) \
		$(top_srcdir)/configure.ac
	$(HELP2MAN) --no-info \
		--name="YubiKey NEO Mode Switcher" \
		--output=$@ $(top_builddir)/ykneo-ccid-modeswitch$(EXEEXT)

# Maintainer rules.

indent:
	indent *.c *.h
	indent *.c *.h

ChangeLog:
	cd $(srcdir) && git2cl > ChangeLog

release:
	@if test -z "$(KEYID)"; then \
		echo "Try this instead:"; \
		echo "  make release KEYID=[PGPKEYID]"; \
		echo "For example:"; \
		echo "  make release KEYID=2117364A"; \
		exit 1; \
	fi
	head -3 $(srcdir)/NEWS | \
		grep -q "Version $(VERSION) .released `date -I`" || \
		(echo 'error: Update date/version in $(srcdir)/NEWS.'; exit 1)
	rm -f $(srcdir)/ChangeLog
	make ChangeLog distcheck
	gpg --detach-sign --default-key $(KEYID) $(PACKAGE)-$(VERSION).tar.gz
	gpg --verify $(PACKAGE)-$(VERSION).tar.gz.sig
	git tag -sm "$(PACKAGE)-$(VERSION)" $(PACKAGE)-$(VERSION)
	git push
	git push --tags
	git add $(PACKAGE)-$(VERSION).tar.gz
	git add $(PACKAGE)-$(VERSION).tar.gz.sig
	git stash
	git checkout gh-pages
	git stash pop
	git mv $(PACKAGE)-$(VERSION).tar.gz releases/
	git mv $(PACKAGE)-$(VERSION).tar.gz.sig releases/
	x=$$(ls -1 releases/*.tar.gz | awk -F\- '{print $$4}' | sed 's/.tar.gz//' | paste -sd ',' -); sed -i -e "2s/\[.*\]/[$$x]/" releases.html
	git add releases.html
	git commit -m "Added tarball for release $(VERSION)."
	git push
	git checkout master
