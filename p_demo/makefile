include $(MAKESHIFT_ROOT)/makeshift-core/makeshift.mk

####################################################################################
# register base modules
####################################################################################

# general modules
$(call _module_root,cloud/par)
$(call _module_root,cloud/gcp)

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

GCP_MAKESHIFT_MODULES=cloud/par cloud/gcp utils/demo

GCP_MAKESHIFT_EXTRA=pipes/metagenomics/p_demo

####################################################################################
# global pipeline parameters
####################################################################################

DRY=F
TOP_WAIT?=T

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

# combo step (steps 1-3)
p_combo: p_rsync p_create_bucket p_run

####################################################################################
# clean-up
####################################################################################

# remove buckets
p_remove_buckets:
	$(MAKE) m=gcp destroy_bucket i=OUT

