include $(MAKESHIFT_ROOT)/makeshift-core/makeshift.mk

####################################################################################
# register base modules
####################################################################################

# general modules
$(call _module_root,cloud/par)
$(call _module_root,cloud/gcp)
$(call _module_root,utils/export)

# demo code here
$(call _module_root,utils/demo)

PIPELINE_NAME?=demo

####################################################################################
# single config file
####################################################################################

c?=examples/demo
$(call _config_root,$c)

####################################################################################
# register other modules
####################################################################################

m?=gcp
$(call _active_module,$(m))

####################################################################################
# define modules that will be uploaded to GCP 
####################################################################################

GCP_MAKESHIFT_MODULES=cloud/par cloud/gcp utils/export utils/demo

GCP_MAKESHIFT_EXTRA=pipes/metagenomics/p_demo

EXPORT_SUBJECT_VARIABLES?=DEMO_SUBJECT_TABLE

####################################################################################
# pipeline steps
####################################################################################

# 1. upload code to cloud
p_rsync:
	$(MAKE) m=gcp rsync

# 2. create output bucket
p_create_bucket:
	$(MAKE) m=gcp create_mount_bucket i=OUT

# 3. call module step
p_run:
	$(MAKE) m=demo demo_subjects

p_export:
	$(MAKE) m=export export_files \
		EXPORT_ID=$(DEMO_SID) \
		EXPORT_VARIABLES="$(EXPORT_SUBJECT_VARIABLES)"

# combo step (steps 1-3)
p_all: p_rsync p_create_bucket p_run p_export

####################################################################################
# export pipeline data
####################################################################################

####################################################################################
# clean-up
####################################################################################

# remove buckets
p_clean:
	$(MAKE) m=gcp destroy_bucket i=OUT

