##############################################################################
# Copyright (c) 2016 Red Hat Inc.
# dradez@redhat.com
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Apache License, Version 2.0
# which accompanies this distribution, and is available at
# http://www.apache.org/licenses/LICENSE-2.0
##############################################################################

export USE_MASTER = ""
export CENTDNLD = http://mirrors.cat.pdx.edu/centos/7.3.1611/isos/x86_64/CentOS-7-x86_64-DVD-1611.iso
export RELEASE = "0"

export RPM_DIST = $(shell rpm -E %dist)

export TACKER_REPO := $(shell awk -F\= '/^tacker_repo/ {print $$2}' variables.sh )
export TACKER_BRANCH := $(shell awk -F\= '/^tacker_branch/ {print $$2}' variables.sh )
export TACKER_COMMIT := $(shell git ls-remote $(TACKER_REPO) $(TACKER_BRANCH) | awk '{print substr($$1,1,7)}')

export TACKERCLIENT_REPO := $(shell awk -F\= '/^tackerclient_repo/ {print $$2}' variables.sh )
export TACKERCLIENT_BRANCH := $(shell awk -F\= '/^tackerclient_branch/ {print $$2}' variables.sh )
export TACKERCLIENT_COMMIT := $(shell git ls-remote $(TACKERCLIENT_REPO) $(TACKERCLIENT_BRANCH) | awk '{print substr($$1,1,7)}')

export CONGRESS_REPO := $(shell awk -F\= '/^congress_repo/ {print $$2}' variables.sh )
export CONGRESS_BRANCH := $(shell awk -F\= '/^congress_branch/ {print $$2}' variables.sh )
export CONGRESS_COMMIT := $(shell git ls-remote $(CONGRESS_REPO) $(CONGRESS_BRANCH) | awk '{print substr($$1,1,7)}')

export NETVPP_VERS := $(shell grep Version $(shell pwd)/rpm_specs/networking-vpp.spec | head -n 1 | awk '{ print $$2 }')
export NETVPP_REPO := $(shell awk -F\= '/^netvpp_repo/ {print $$2}' variables.sh )
export NETVPP_BRANCH := $(shell awk -F\= '/^netvpp_branch/ {print $$2}' variables.sh )
export NETVPP_COMMIT := $(shell git ls-remote $(NETVPP_REPO) $(NETVPP_BRANCH) | awk '{print substr($$1,1,7)}')

export RELRPMVERS = $(shell grep Version $(shell pwd)/rpm_specs/opnfv-apex-release.spec | head -n 1 | awk '{ print $$2 }')
export RPMVERS = $(shell grep Version $(shell pwd)/rpm_specs/opnfv-apex.spec | head -n 1 | awk '{ print $$2 }')

export BUILD_ROOT = $(shell pwd)
export BUILD_DIR = $(shell dirname $$(pwd))/.build
export CACHE_DIR = $(shell dirname $$(pwd))/.cache
export PATCHES_DIR = $(BUILD_ROOT)/patches
export QUAGGA_BUILD_DIR = $(BUILD_DIR)/quagga_build_dir
export QUAGGA_RPMS_DIR = $(QUAGGA_BUILD_DIR)/rpmbuild
export RPM_DIR_ARGS = -D '_topdir $(BUILD_DIR)' -D '_builddir $(BUILD_DIR)' -D '_sourcedir $(BUILD_DIR)' -D '_rpmdir $(BUILD_DIR)' -D '_specdir $(BUILD_DIR)' -D '_srcrpmdir $(BUILD_DIR)'

export RPMREL = $(BUILD_DIR)/noarch/opnfv-apex-release-$(RPMVERS)-$(shell echo ${RELEASE} | tr -d '_-').noarch.rpm
export RPMCOM = $(BUILD_DIR)/noarch/opnfv-apex-common-$(RPMVERS)-$(shell echo ${RELEASE} | tr -d '_-').noarch.rpm
export RPMUDR = $(BUILD_DIR)/noarch/opnfv-apex-undercloud-$(RPMVERS)-$(shell echo ${RELEASE} | tr -d '_-').noarch.rpm
export RPMODL = $(BUILD_DIR)/noarch/opnfv-apex-$(RPMVERS)-$(shell echo ${RELEASE} | tr -d '_-').noarch.rpm
export RPMONO = $(BUILD_DIR)/noarch/opnfv-apex-onos-$(RPMVERS)-$(shell echo ${RELEASE} | tr -d '_-').noarch.rpm
export ISO = $(BUILD_DIR)/release/OPNFV-CentOS-7-x86_64-${RELEASE}.iso
export CENTISO = $(BUILD_DIR)/$(shell basename $(CENTDNLD))

.PHONY: all
all: iso

.PHONY: clean
clean:
	rm -rf $(BUILD_DIR)

.PHONY: clean-cache
clean-cache:
	rm -rf $(CACHE_DIR)

# ONOS integration did not get updated for Danube,
# removing it from the default builds

.PHONY: images
images: undercloud overcloud-full overcloud-opendaylight # overcloud-onos

.PHONY: rpms
rpms: common-rpm undercloud-rpm opendaylight-rpm # onos-rpm

.PHONY: rpms-check
rpms-check: release-rpm-check common-rpm-check undercloud-rpm-check opendaylight-rpm-check # onos-rpm-check

.PHONY: rpms-clean
rpms-clean:
	rm -rf $(BUILD_DIR)/noarch
	rm -rf $(BUILD_DIR)/BUILDROOT

$(BUILD_DIR)/opnfv-apex-release.tar.gz:
	mkdir -p $(BUILD_DIR)
	pushd ../ && git archive --format=tar.gz --prefix=opnfv-apex-release-$(RELRPMVERS)/ HEAD > $(BUILD_DIR)/opnfv-apex-release.tar.gz

.PHONY: release-rpm-check
release-rpm-check: $(BUILD_DIR)/opnfv-apex-release.tar.gz
	rpmbuild --clean -bi -bl rpm_specs/opnfv-apex-release.spec $(RPM_DIR_ARGS) -D "release $(shell echo $(RELEASE) | tr -d '_-')"

.PHONY: release-rpm
release-rpm: $(BUILD_DIR)/opnfv-apex-release.tar.gz $(RPMREL)

$(RPMREL):
	@echo "Building the Apex Release RPM"
	# build the release RPM
	rpmbuild --clean -ba rpm_specs/opnfv-apex-release.spec $(RPM_DIR_ARGS) -D "release $(shell echo $(RELEASE) | tr -d '_-')"

$(BUILD_DIR)/opnfv-apex-common.tar.gz:
	pushd ../ && git archive --format=tar.gz --prefix=opnfv-apex-common-$(RPMVERS)/ HEAD > $(BUILD_DIR)/opnfv-apex-common.tar.gz

.PHONY: common-rpm-check
common-rpm-check: $(BUILD_DIR)/opnfv-apex-common.tar.gz
	rpmbuild --clean -bi -bl rpm_specs/opnfv-apex-common.spec $(RPM_DIR_ARGS) -D "release $(shell echo $(RELEASE) | tr -d '_-')"

.PHONY: common-rpm
common-rpm: $(BUILD_DIR)/opnfv-apex-common.tar.gz $(RPMCOM)

$(RPMCOM):
	@echo "Building the Apex Common RPM"
	# build the common RPM
	rpmbuild --clean -ba rpm_specs/opnfv-apex-common.spec $(RPM_DIR_ARGS) -D "release $(shell echo $(RELEASE) | tr -d '_-')"

##################
#  PYTHON TESTS  #
##################

.PHONY: python-tests
python-tests:
	# clean previous coverage data
	rm -rf ../tests/.coverage
	rm -rf ../tests/htmlcov
	# run nose tests
	cd ../tests && PYTHONPATH=../lib/python/ nosetests-3.4 . --with-coverage --cover-package apex --cover-package apex_python_utils --cover-html --cover-min-percentage 90

	# generate reports
	cd ../tests && coverage3 report --include '*lib/python/*' -m

#######################
#  PYTHON PEP8 CHECK  #
#######################

.PHONY: python-pep8-check
python-pep8-check:
	pep8 ../lib/python
	pep8 ../tests

###############
#  TACKER     #
###############

$(BUILD_DIR)/openstack-tacker.tar.gz:
	@echo "Preparing the Tacker RPM prerequisites"
	git clone $(TACKER_REPO) -b $(TACKER_BRANCH) $(BUILD_DIR)/openstack-tacker-2016.2
	cp rpm_specs/openstack-tacker-server.service $(BUILD_DIR)/openstack-tacker-2016.2
	tar czf $(BUILD_DIR)/openstack-tacker.tar.gz -C $(BUILD_DIR) openstack-tacker-2016.2

.PHONY: tacker-rpm
tacker-rpm: $(BUILD_DIR)/openstack-tacker.tar.gz $(BUILD_DIR)/noarch/openstack-tacker-2016.2-1.git$(TACKER_COMMIT).noarch.rpm

$(BUILD_DIR)/noarch/openstack-tacker-2016.2-1.git$(TACKER_COMMIT).noarch.rpm:
	@echo "Building the Tacker RPM"
	rpmbuild --clean -ba --target noarch rpm_specs/openstack-tacker.spec $(RPM_DIR_ARGS) -D 'git .git$(TACKER_COMMIT)'

#################
#  TACKERCLIENT #
#################

$(BUILD_DIR)/python-tackerclient.tar.gz:
	@echo "Preparing the TackerClient RPM prerequisites"
	git clone $(TACKERCLIENT_REPO) -b $(TACKERCLIENT_BRANCH) $(BUILD_DIR)/python-tackerclient-2016.2
	tar czf $(BUILD_DIR)/python-tackerclient.tar.gz -C $(BUILD_DIR) python-tackerclient-2016.2

.PHONY: tackerclient-rpm
tackerclient-rpm: $(BUILD_DIR)/python-tackerclient.tar.gz $(BUILD_DIR)/noarch/python-tackerclient-2016.2-1.git$(TACKERCLIENT_COMMIT).noarch.rpm

$(BUILD_DIR)/noarch/python-tackerclient-2016.2-1.git$(TACKERCLIENT_COMMIT).noarch.rpm:
	@echo "Building the TackerClient RPM"
	rpmbuild --clean -ba --target noarch rpm_specs/python-tackerclient.spec $(RPM_DIR_ARGS) -D 'git .git$(TACKERCLIENT_COMMIT)'

###############
#  CONGRESS   #
###############

.PHONY: congress-clean
congress-clean:
	@rm -rf $(BUILD_DIR)/openstack-congress-2016.2
	@rm -f $(BUILD_DIR)/openstack-congress.tar.gz

$(BUILD_DIR)/openstack-congress.tar.gz:
	@echo "Preparing the Congress RPM prerequisites"
	git clone $(CONGRESS_REPO) -b $(CONGRESS_BRANCH) $(BUILD_DIR)/openstack-congress-2016.2
	cp rpm_specs/openstack-congress-server.service $(BUILD_DIR)/openstack-congress-2016.2
	tar czf $(BUILD_DIR)/openstack-congress.tar.gz -C $(BUILD_DIR) openstack-congress-2016.2

.PHONY: congress-rpm
congress-rpm: $(BUILD_DIR)/noarch/openstack-congress-2016.2-1.git$(CONGRESS_COMMIT).noarch.rpm

$(BUILD_DIR)/noarch/openstack-congress-2016.2-1.git$(CONGRESS_COMMIT).noarch.rpm: $(BUILD_DIR)/openstack-congress.tar.gz
	@echo "Building the Congress RPM"
	rpmbuild --clean -ba --target noarch rpm_specs/openstack-congress.spec $(RPM_DIR_ARGS) -D 'git .git$(CONGRESS_COMMIT)'

##################
# NETWORKING-VPP #
##################

$(BUILD_DIR)/python-networking-vpp.tar.gz:
	@echo "Preparing the networking-vpp RPM prerequisites"
	git clone $(NETVPP_REPO) $(BUILD_DIR)/python-networking-vpp-$(NETVPP_VERS)
	tar czf $(BUILD_DIR)/python-networking-vpp.tar.gz -C $(BUILD_DIR) python-networking-vpp-$(NETVPP_VERS)

.PHONY: networking-vpp-rpm
networking-vpp-rpm: $(BUILD_DIR)/noarch/python-networking-vpp-$(NETVPP_VERS)-1.git$(NETVPP_COMMIT)$(RPM_DIST).noarch.rpm

$(BUILD_DIR)/noarch/python-networking-vpp-$(NETVPP_VERS)-1.git$(NETVPP_COMMIT)$(RPM_DIST).noarch.rpm: $(BUILD_DIR)/python-networking-vpp.tar.gz
	@echo "Building the Networking VPP RPM"
	rpmbuild --clean -ba --target noarch rpm_specs/networking-vpp.spec $(RPM_DIR_ARGS) -D 'git .git$(NETVPP_COMMIT)'

###############
#  UNDERCLOUD #
###############

.PHONY: undercloud
undercloud: $(BUILD_DIR)/undercloud.qcow2

$(BUILD_DIR)/undercloud.qcow2: tackerclient-rpm
	@echo "Building the Apex Undercloud Image"
	@./undercloud.sh

$(BUILD_DIR)/opnfv-apex-undercloud.tar.gz: $(BUILD_DIR)/undercloud.qcow2
	@echo "Preparing the Apex Undercloud RPM prerequisites"
	pushd ../ && git archive --format=tar --prefix=opnfv-apex-undercloud-$(RPMVERS)/ HEAD > $(BUILD_DIR)/opnfv-apex-undercloud.tar
	tar -rf $(BUILD_DIR)/opnfv-apex-undercloud.tar \
               --xform="s:.*undercloud.qcow2:opnfv-apex-undercloud-$(RPMVERS)/build/undercloud.qcow2:" $(BUILD_DIR)/undercloud.qcow2
	gzip -f $(BUILD_DIR)/opnfv-apex-undercloud.tar

.PHONY: undercloud-rpm-check
undercloud-rpm-check: $(BUILD_DIR)/opnfv-apex-undercloud.tar.gz
	rpmbuild --clean -bi -bl rpm_specs/opnfv-apex-undercloud.spec $(RPM_DIR_ARGS) -D "release $(shell echo $(RELEASE) | tr -d '_-')"

.PHONY: undercloud-rpm
undercloud-rpm: $(BUILD_DIR)/opnfv-apex-undercloud.tar.gz $(RPMUDR)

$(RPMUDR):
	@echo "Building the Apex Undercloud RPM"
	rpmbuild --clean -ba rpm_specs/opnfv-apex-undercloud.spec $(RPM_DIR_ARGS) -D "release $(shell echo $(RELEASE) | tr -d '_-')"

###############
#  OVERCLOUD  #
###############

.PHONY: overcloud-full
overcloud-full: $(BUILD_DIR)/overcloud-full.qcow2

$(BUILD_DIR)/overcloud-full.qcow2: congress-rpm tacker-rpm networking-vpp-rpm
	@echo "Building the Apex Base Overcloud Image"
	@./overcloud-full.sh

###############
#    ODL      #
###############

.PHONY: overcloud-opendaylight
overcloud-opendaylight: $(BUILD_DIR)/overcloud-full-opendaylight.qcow2

$(BUILD_DIR)/overcloud-full-opendaylight.qcow2: $(BUILD_DIR)/overcloud-full.qcow2
	@echo "Building the Apex OpenDaylight Overcloud Image"
	@./overcloud-opendaylight.sh

$(BUILD_DIR)/opnfv-apex.tar.gz: $(BUILD_DIR)/overcloud-full-opendaylight.qcow2
	tar -czf $(BUILD_DIR)/opnfv-apex.tar.gz --xform="s:.*overcloud-full-opendaylight.qcow2:opnfv-apex-$(RPMVERS)/build/overcloud-full-opendaylight.qcow2:" $(BUILD_DIR)/overcloud-full-opendaylight.qcow2

.PHONY: opendaylight-rpm-check
opendaylight-rpm-check: $(BUILD_DIR)/opnfv-apex.tar.gz
	rpmbuild --clean -bi -bl rpm_specs/opnfv-apex.spec $(RPM_DIR_ARGS) -D "release $(shell echo $(RELEASE) | tr -d '_-')"

.PHONY: opendaylight-rpm
opendaylight-rpm: $(BUILD_DIR)/opnfv-apex.tar.gz $(RPMODL)

$(RPMODL):
	@echo "Building the Apex OpenDaylight RPM"
	# build the overcloud RPM
	rpmbuild --clean -ba rpm_specs/opnfv-apex.spec $(RPM_DIR_ARGS) -D "release $(shell echo $(RELEASE) | tr -d '_-')"

###############
#    ONOS     #
###############

.PHONY: overcloud-onos
overcloud-onos: $(BUILD_DIR)/overcloud-full-onos.qcow2

$(BUILD_DIR)/overcloud-full-onos.qcow2: $(BUILD_DIR)/overcloud-full.qcow2
	@echo "Building the Apex ONOS Overcloud Image"
	@./overcloud-onos.sh

$(BUILD_DIR)/opnfv-apex-onos.tar.gz: $(BUILD_DIR)/overcloud-full-onos.qcow2
	tar -czf $(BUILD_DIR)/opnfv-apex-onos.tar.gz --xform="s:.*overcloud-full-onos.qcow2:opnfv-apex-onos-$(RPMVERS)/build/overcloud-full-onos.qcow2:" $(BUILD_DIR)/overcloud-full-onos.qcow2

.PHONY: onos-rpm-check
onos-rpm-check: $(BUILD_DIR)/opnfv-apex-onos.tar.gz
	rpmbuild --clean -bi -bl rpm_specs/opnfv-apex-onos.spec $(RPM_DIR_ARGS) -D "release $(shell echo $(RELEASE) | tr -d '_-')"

.PHONY: onos-rpm
onos-rpm: $(BUILD_DIR)/opnfv-apex-onos.tar.gz $(RPMONO)

$(RPMONO):
	@echo "Building the Apex ONOS RPM"
	# build the overcloud RPM
	rpmbuild --clean -ba rpm_specs/opnfv-apex-onos.spec $(RPM_DIR_ARGS) -D "release $(shell echo $(RELEASE) | tr -d '_-')"

###############
#    ISO      #
###############

$(CENTISO):
	curl $(CENTDNLD) -z $(CENTISO) -o $(CENTISO) --verbose --silent --location

iso-clean:
	@rm -Rf $(BUILD_DIR)/centos
	@rm -Rf $(BUILD_DIR)/release
	@rm -f $(ISO)

.PHONY: mount-centiso umount-centiso
mount-centiso: $(CENTISO)
	@echo "Mounting CentOS ISO in $(CENTDIR)"
	@mkdir -p $(CENTDIR)
	@fuseiso $(CENTISO) $(CENTDIR)

umount-centiso:
	@set +e
	@echo "Unmounting CentOS ISO from $(CENTDIR)"
	@fusermount -u $(CENTDIR)
	@rmdir $(CENTDIR)
	@set -e

.PHONY: iso
iso:	iso-clean images rpms $(CENTISO)
	@echo "Building the Apex ISO"
	@mkdir $(BUILD_DIR)/centos $(BUILD_DIR)/release
	cd $(BUILD_DIR)/centos && bsdtar -xf ../$(shell basename $(CENTISO))
	# modify the installer iso's contents
	@chmod -R u+w $(BUILD_DIR)/centos
	@cp -f isolinux.cfg $(BUILD_DIR)/centos/isolinux/isolinux.cfg
	@ln $(RPMCOM) $(BUILD_DIR)/centos/Packages
	@ln $(RPMUDR) $(BUILD_DIR)/centos/Packages
	@ln $(RPMODL) $(BUILD_DIR)/centos/Packages
	# add packages to the centos packages
	cd $(BUILD_DIR)/centos/Packages && yumdownloader openvswitch openstack-tripleo jq python34 python34-libs python34-yaml python34-setuptools ipxe-roms-qemu
	cd $(BUILD_DIR)/centos/Packages && curl -O https://radez.fedorapeople.org/python34-markupsafe-0.23-9.el7.centos.x86_64.rpm
	cd $(BUILD_DIR)/centos/Packages && curl -O https://radez.fedorapeople.org/python3-jinja2-2.8-5.el7.centos.noarch.rpm
	cd $(BUILD_DIR)/centos/Packages && curl -O http://artifacts.opnfv.org/apex/dependencies/python3-ipmi-0.3.0-1.noarch.rpm
	# regenerate yum repo data
	@echo "Generating new yum metadata"
	createrepo --update -g $(BUILD_ROOT)/c7-opnfv-x86_64-comps.xml $(BUILD_DIR)/centos
	# build the iso
	@echo "Building OPNFV iso"
	mkisofs -b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table -V "OPNFV CentOS 7 x86_64" -R -J -v -T -o $(ISO) $(BUILD_DIR)/centos
	isohybrid $(ISO)
	@printf "\n\nISO is built at $(ISO)\n\n"

####################
#  python3-jinja2  #
####################

.PHONY: python3-jinja2
python3-jinja2: python3-markupsafe
	cd $(BUILD_DIR) \
	&& curl -O -L artifacts.opnfv.org/apex/dependencies/python-jinja2-2.8-5.fc24.src.rpm \
	&& rpm2cpio python-jinja2-2.8-5.fc24.src.rpm | cpio -idmv \
	&& sed -i 's/python3-devel/python34-devel/' python-jinja2.spec \
	&& sed -i 's/python3-setuptools/python34-setuptools/' python-jinja2.spec \
	&& sed -i 's/python3-pytest/python34-pytest/' python-jinja2.spec \
	&& sed -i 's/python3-markupsafe/python34-markupsafe/' python-jinja2.spec \
	&& rpmbuild -ba python-jinja2.spec $(RPM_DIR_ARGS) -D "with_python3 1"

########################
#  python3-markupsafe  #
########################

.PHONY: python3-markupsafe
python3-markupsafe:
	cd $(BUILD_DIR) \
	&& curl -O -L artifacts.opnfv.org/apex/dependencies/python-markupsafe-0.23-9.fc24.src.rpm \
	&& rpm2cpio python-markupsafe-0.23-9.fc24.src.rpm | cpio -idmv \
	&& sed -i 's/python3-devel/python34-devel/' python-markupsafe.spec \
	&& sed -i 's/python3-setuptools/python34-setuptools/' python-markupsafe.spec \
	&& sed -i 's/python3-pytest/python34-pytest/' python-markupsafe.spec \
	&& sed -i 's/python3-markupsafe/python34-markupsafe/' python-markupsafe.spec \
	&& rpmbuild -ba python-markupsafe.spec $(RPM_DIR_ARGS) -D "with_python3 1"

##################
#  Quagga Clean  #
##################
.PHONY: quagga-clean
quagga-clean:
	@rm -rf $(QUAGGA_BUILD_DIR)
	@sudo yum -y remove zrpc* quagga* c-capnproto* thrift*

#################
#  Quagga+ZRPC  #
#################
.PHONY: quagga-zrpc
quagga-zrpc: quagga-clean thrift-rpm capnproto-rpm quagga-rpm zrpc-rpm

##########
#  ZRPC  #
##########
.PHONY: zrpc-rpm
zrpc-rpm: quagga-rpm $(QUAGGA_RPMS_DIR)/zrpcd-%.x86_64.rpm

$(QUAGGA_RPMS_DIR)/zrpcd-%.x86_64.rpm:
	@echo "Building ZRPC RPM"
	@./build_quagga.sh -a zrpc

############
#  Quagga  #
############
.PHONY: quagga-rpm
quagga-rpm: $(QUAGGA_RPMS_DIR)/RPMS/x86_64/quagga-1.1.0_%.el7.centos.x86_64.rpm

$(QUAGGA_RPMS_DIR)/RPMS/x86_64/quagga-1.1.0_%.el7.centos.x86_64.rpm:
	@echo "Building Quagga RPM"
	@./build_quagga.sh -a quagga

###############
#  Capnproto  #
###############
.PHONY: capnproto-rpm
capnproto-rpm: $(QUAGGA_RPMS_DIR)/RPMS/x86_64/c-capnproto-%.x86_64.rpm

$(QUAGGA_RPMS_DIR)/RPMS/x86_64/c-capnproto-%.x86_64.rpm:
	@echo "Building capnproto RPMs"
	@./build_quagga.sh -a capnproto

############
#  Thrift  #
############

.PHONY: thrift-rpm
thrift-rpm: $(QUAGGA_RPMS_DIR)/RPMS/x86_64/thrift-%.x86_64.rpm

$(QUAGGA_RPMS_DIR)/RPMS/x86_64/thrift-%.x86_64.rpm:
	@echo "Building Thrift RPMs"
	@./build_quagga.sh -a thrift
