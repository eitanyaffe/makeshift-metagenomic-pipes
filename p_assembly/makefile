include $(MAKESHIFT_ROOT)/makeshift-core/makeshift.mk

####################################################################################
####################################################################################
# PIPELINE DEFS
####################################################################################
####################################################################################

PIPELINE_NAME?=assembly

####################################################################################
# register base modules
####################################################################################

$(call _module_root,cloud/par)
$(call _module_root,cloud/gcp)

####################################################################################
# set config file
####################################################################################

c?=examples/pp_test
$(call _config_root,$c)

####################################################################################
# register other modules
####################################################################################

# input: populate bucket with fastq files 
$(call _module_root,metagenomics/input)

# sgcc: genetic signature distance matrix
$(call _module_root,metagenomics/sgcc)

# lib pre-process
$(call _module_root,metagenomics/libs)

# assembly
$(call _module_root,metagenomics/assembly)

# genes
$(call _module_root,metagenomics/genes)

# map
$(call _module_root,metagenomics/map)

# export
$(call _module_root,utils/export)

####################################################################################
# set module
####################################################################################

m?=gcp
$(call _active_module,$(m))

####################################################################################
# define modules that are uploaded to GCP 
####################################################################################

GCP_MAKESHIFT_MODULES=\
cloud/par cloud/gcp \
utils/export \
metagenomics/input \
metagenomics/sgcc \
metagenomics/libs \
metagenomics/assembly \
metagenomics/genes \
metagenomics/map

GCP_MAKESHIFT_EXTRA=pipes/metagenomics/p_assembly

####################################################################################
####################################################################################
# PIPELINE STEPS
####################################################################################
####################################################################################

####################################################################################
# utility steps
####################################################################################

# compile c++ programs into the binary bucket
p_init: freeze_module_versions
	$(MAKE) m=gcp create_mount_bucket i=B
	$(MAKE) m=libs libs_init

# upload config and code to cloud 
p_rsync:
	@$(MAKE) m=gcp rsync

# create input/output/bin buckets
p_output_bucket:
	$(MAKE) m=gcp create_mount_bucket i=OUT

# examine bucket space usage of project
p_space:
	$(MAKE) m=gcp gcp_du_project

####################################################################################
# signature based genetic comparison
####################################################################################

# sgcc for all libraries
d_sgcc:
	@$(MAKE) m=sgcc top_sgcc

####################################################################################
# libs
####################################################################################

# libraries for single assembly
d_lib:
	@$(MAKE) m=libs top_lib

# libraries for all assemblies
d_libs:
	@$(MAKE) m=libs top_libs

# collect stats (read count and yields of various lib steps)
p_libs_select:
	@$(MAKE) m=libs libs_stats libs_dup_stats libs_select

p_libs_plot:
	@$(MAKE) m=libs libs_plot

####################################################################################
# assembly
####################################################################################

# single assembly
d_assembly:
	@$(MAKE) m=assembly top_assembly

# all assemblies
d_assembly_all:
	@$(MAKE) m=assembly top_assembly_all

# assembly stats
p_assembly_stats:
	@$(MAKE) m=assembly assembly_stats plot_stats

####################################################################################
# map
####################################################################################

# map specific assembly
d_map:
	@$(MAKE) m=map top_map

# map all assemblies
d_map_all:
	@$(MAKE) m=map top_map_all

p_map_stats:
	@$(MAKE) m=map map_stats plot_stats

####################################################################################
# genes
####################################################################################

# predict genes
d_genes:
	@$(MAKE) m=genes top_genes

# predict genes over all assemblies
d_genes_all:
	@$(MAKE) m=genes top_genes_all

####################################################################################
# gene coverage
####################################################################################

# cov, single assembly
d_genes_cov:
	@$(MAKE) m=genes top_genes_cov

# cov, all assemblies
d_genes_cov_all:
	@$(MAKE) m=genes top_genes_cov_all

####################################################################################
# advanced features
####################################################################################

# mount pipeline buckets under user home directory
# (call outside docker container)
local_mount:
	$(MAKE) m=gcp mount_buckets_home

####################################################################################
# !Danger Zone!
####################################################################################

# remove output bucket
p_remove_buckets:
	$(MAKE) m=gcp destroy_bucket i=OUT
	$(MAKE) m=gcp destroy_bucket i=B

####################################################################################
####################################################################################
# PIPELINE MAIN
####################################################################################
####################################################################################

# once per pipeline
p_init_pipeline: p_init p_rsync

# once per project
p_init_project: p_output_bucket

# from reads to assembly and genes
p_run_basic: \
d_sgcc \
d_libs \
p_libs_select \
d_assembly_all \
d_map_all \
d_genes_all \
d_genes_cov_all

T?=p_run_basic
top:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(TOP_MANAGER_DIR) \
		PAR_MODULE=par \
		PAR_NAME=top \
		PAR_ODIR_VAR=TOP_WORK_DIR \
		PAR_TARGET=$(T) \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

####################################################################################
# plots
####################################################################################

p_basic_plots: \
p_libs_plot \
p_assembly_stats \
p_map_stats

####################################################################################
# export pipeline data
####################################################################################

EXPORT_ASSEMBLY_VARIABLES?=\
$(SGCC_EXPORT_VARS) \
$(LIBS_EXPORT_VARS) \
$(ASSEMBLY_EXPORT_VARS) \
$(GENES_EXPORT_VARS) \
$(MAP_EXPORT_VARS)

# please specify ASSEMBLY_ID
p_export_assembly:
	$(MAKE) m=export export_files \
		EXPORT_ID=$(ASSEMBLY_ID) \
		EXPORT_VARIABLES="$(EXPORT_ASSEMBLY_VARIABLES)"
