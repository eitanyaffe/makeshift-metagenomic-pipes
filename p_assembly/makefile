include $(MAKESHIFT_ROOT)/makeshift-core/makeshift.mk

####################################################################################
# register base modules
####################################################################################

$(call _module_root,cloud/par)
$(call _module_root,cloud/gcp)

####################################################################################
# single config file
####################################################################################

#c=arsenic
#c=test
#c=metagenomics/c60

c?=examples/SRA_example
#$(call _config_root,metagenomics/$c)
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

# map"
$(call _module_root,metagenomics/map)

m?=gcp
$(call _active_module,$(m))

####################################################################################
# define modules that will be uploaded to GCP 
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
# global pipeline parameters
####################################################################################

DRY=F
TOP_WAIT?=T

####################################################################################
# buckets
####################################################################################

# create output bucket
p_create_buckets:
	$(MAKE) m=gcp make_bucket i=IN
	$(MAKE) m=gcp make_bucket i=OUT

pm:
	$(MAKE) m=gcp mount_buckets

pmx:
	$(MAKE) m=gcp mount_buckets GCP_GCSFUSE_IMPLICIT_DIRS=F

# mount under home
pmh:
	$(MAKE) m=gcp mount_buckets_home

####################################################################################
# prepare input fastq files
####################################################################################

# download SRA data to input bucket using INPUT_SRA_TABLE
p_download_sra:
	$(MAKE) m=input d_input_sra

# user-specific prepping of the input table
p_create_lib_table:
	$(MAKE) config_create_tables

####################################################################################
# compile c++ code
####################################################################################

p_init:
	@$(MAKE) m=libs libs_init

####################################################################################
# signature based genetic comparison
####################################################################################

d_sgcc:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(SGCC_INFO_DIR) \
		PAR_MODULE=sgcc \
		PAR_NAME=sgcc \
		PAR_ODIR_VAR=SGCC_INFO_DIR \
		PAR_TARGET=s_sgcc \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_NOTIFY_MAX_LEVEL=1 \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

####################################################################################
# libs
####################################################################################

d_lib:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(LIB_INFO_DIR) \
		PAR_MODULE=libs \
		PAR_NAME=lib_top \
		PAR_ODIR_VAR=LIB_INFO_DIR \
		PAR_TARGET=s_lib \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

d_libs:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(LIBS_MULTI_DIR) \
		PAR_MODULE=libs \
		PAR_NAME=libs_top \
		PAR_ODIR_VAR=LIBS_MULTI_DIR \
		PAR_TARGET=s_libs \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

# plot stats (read count and yields of various lib steps)
p_libs_select:
	@$(MAKE) m=libs libs_stats libs_dup_stats libs_select plot_stats

p_libs_plot:
	@$(MAKE) m=libs libs_plot

####################################################################################
# assembly
####################################################################################

d_assembly:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(ASSEMBLY_INFO_DIR) \
		PAR_MODULE=assembly \
		PAR_NAME=top_assembly \
		PAR_ODIR_VAR=ASSEMBLY_INFO_DIR \
		PAR_TARGET=s_assembly \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

d_assemblies:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(ASSEMBLY_MULTI_DIR) \
		PAR_MODULE=assembly \
		PAR_NAME=top_assemblies \
		PAR_ODIR_VAR=ASSEMBLY_MULTI_DIR \
		PAR_TARGET=s_assemblies \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

# assembly stats
p_assembly_stats:
	@$(MAKE) m=assembly assembly_stats plot_stats

####################################################################################
# map
####################################################################################

# libs of specific assembly
d_map:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(MAP_SET_DIR) \
		PAR_MODULE=map \
		PAR_NAME=map_set \
		PAR_ODIR_VAR=MAP_SET_DIR \
		PAR_TARGET=s_map_set \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

# map all assemblies
d_map_all:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(MAP_MULTI_DIR) \
		PAR_MODULE=map \
		PAR_NAME=map_all \
		PAR_ODIR_VAR=MAP_MULTI_DIR \
		PAR_TARGET=s_map_all \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

p_map_stats:
	@$(MAKE) m=map map_stats plot_stats

####################################################################################
# genes
####################################################################################

d_genes:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(GENES_INFO_DIR) \
		PAR_MODULE=genes \
		PAR_NAME=top_genes_single \
		PAR_ODIR_VAR=GENES_INFO_DIR \
		PAR_TARGET=s_genes \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

# all assemblies
d_genes_all:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(GENES_MULTI_DIR) \
		PAR_MODULE=genes \
		PAR_NAME=top_genes \
		PAR_ODIR_VAR=GENES_MULTI_DIR \
		PAR_TARGET=s_genes_multi \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

####################################################################################
# gene coverage
####################################################################################

# cov, single assembly
d_genes_cov:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(GENES_COVERAGE_INFO_DIR) \
		PAR_MODULE=genes \
		PAR_NAME=top_genes_cov \
		PAR_ODIR_VAR=GENES_COVERAGE_INFO_DIR \
		PAR_TARGET=s_genes_cov_mat \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

# cov, all assemblies
d_genes_cov_all:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(GENES_COVERAGE_MULTI_DIR) \
		PAR_MODULE=genes \
		PAR_NAME=top_genes_cov \
		PAR_ODIR_VAR=GENES_COVERAGE_MULTI_DIR \
		PAR_TARGET=s_genes_cov_multi \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

####################################################################################
# export
####################################################################################

p_export:
	$(MAKE) m=sgcc sgcc_export
	$(MAKE) m=libs libs_export
	$(MAKE) m=assembly assembly_export
	$(MAKE) m=map map_export
	$(MAKE) m=genes genes_export

d_export:
	$(MAKE) m=par par \
		PAR_WORK_DIR=$(BASE_EXPORT_DIR) \
		PAR_MODULE=par \
		PAR_NAME=export \
		PAR_ODIR_VAR=BASE_EXPORT_DIR \
		PAR_TARGET=p_export \
		PAR_DISK_GB=500 \
		PAR_PREEMTIBLE=0 \
		PAR_WAIT=$(TOP_WAIT) \
		PAR_MAKEFLAGS="$(PAR_MAKEOVERRIDES)"

####################################################################################
# combo steps
####################################################################################

p_all: \
rsync \
p_download_sra \
d_sgcc \
d_libs p_libs_select p_libs_plot \
d_assemblies p_assembly_stats \
d_map_all p_map_stats \
d_genes_all \
d_genes_cov_all d_export

####################################################################################
# utility steps
####################################################################################

# bucket space usage
p_space:
	$(MAKE) m=gcp gcp_du_project
