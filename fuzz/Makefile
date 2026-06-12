# https://stackoverflow.com/questions/18136918/how-to-get-current-relative-directory-of-your-makefile
THIS_DIR := $(dir $(realpath $(lastword $(MAKEFILE_LIST))))

# https://llvm.org/docs/LibFuzzer.html

# name of the fuzz binary (caller needs to set)
FUZZBIN ?= fuzzbinary
# more options to pass to the fuzz binary
# see `make help` or https://llvm.org/docs/LibFuzzer.html
OPTIONS ?=

# token dictionary
DICT ?= 1
DICT_YAML ?= yaml.dict
DICT_JSON ?= json.dict
ifneq ($(DICT),0)
OPTS_YAML_ += -dict=$(DICT_YAML)
OPTS_JSON_ += -dict=$(DICT_JSON)
endif

# whether to merge cases into representative cases
# (merged cases are written into the first corpus directory)
MERGE ?= 0
OPTS_ += -merge=$(MERGE)
# merge control file
ifneq ($(MERGE),0)
OPTS_YAML_ += -merge_control_file=mergefile-yaml-$(shell basename $(FUZZBIN)).dat
OPTS_JSON_ += -merge_control_file=mergefile-json-$(shell basename $(FUZZBIN)).dat
endif

# number of threads
WORKERS ?= $(shell echo $$(( $(shell nproc) / 2 )))
OPTS_ += -workers=$(WORKERS)
# how many errors to find (or hit duration)
JOBS ?= 20
OPTS_ += -fork=$(JOBS)
# re-read periodically to check for new inputs
RELOAD ?= 1
OPTS_ += -reload=$(RELOAD)
# fuzzing duration in seconds. 0 means unlimited
DURATION ?= 3600
OPTS_ += -max_total_time=$(DURATION)
# max length of a fuzzing case. 0 means guess the size
MAXLEN ?= 0
OPTS_ += -max_len=$(MAXLEN)
# rate at which length increases (small==faster, default is 100)
# 0 means immediately maxlen
LENCTRL ?= 100
OPTS_ += -len_control=$(LENCTRL)
# in seconds. timeout for an individual case
TIMEOUT ?= 10
OPTS_ += -timeout=$(TIMEOUT)
# whether to print stats at the end
STATS ?= 0
OPTS_ += -print_final_stats=$(STATS)
# 0 means generate the seed randomly
SEED ?= 0
OPTS_ += -seed=$(SEED)

# corpus directories
# the first directory is the destination when merging!
CORPUS_YAML_ALL := \
	yaml_merged/ \
	yaml_fails/ \
	yaml_test_suite/ \
	yaml_rapidyamldump/
CORPUS_JSON_ALL := \
	json_merged/ \
	json_fails/ \
	json_rapidyamldump/
# can be set by the caller
CORPUS_YAML ?= $(CORPUS_YAML_ALL)
CORPUS_JSON ?= $(CORPUS_JSON_ALL)
# where to place failure cases
# note the trailing slash! it is neeeded for directories
FAIL_DIR_YAML ?= yaml_fails/
FAIL_DIR_JSON ?= json_fails/

# keep going on failure case?
IGNORE_OOMS ?= 1
IGNORE_TIMEOUTS ?= 1
IGNORE_CRASHES ?= 1
OPTS_ += -ignore_ooms=$(IGNORE_OOMS)
OPTS_ += -ignore_timeouts=$(IGNORE_TIMEOUTS)
OPTS_ += -ignore_crashes=$(IGNORE_CRASHES)


#----------------------------

.PHONY: help
help:
	@echo "Targets:"
	@echo
	@echo "    yaml  - fuzz yaml"
	@echo "    json  - fuzz json"
	@echo "    stop  - stop fuzzing session (use from different console, or Ctrl-Z before 'make stop')"
	@echo "    check - verify fuzz executable"
	@echo
	@echo "Examples:"
	@echo
	@echo "    make allbin DURATION=100"
	@echo "    make FUZZBIN=ryml-fuzz-yaml_tree DURATION=100"
	@echo
	@echo "Fuzz help:"
	@echo
	$(FUZZBIN) -help=1 2>&1


.PHONY: allbin
allbin:
	set -xe && \
	  for exe in bin/*yaml* ; do $(MAKE) yaml FUZZBIN=$$exe ; done
	set -xe && \
	  for exe in bin/*json* ; do $(MAKE) json FUZZBIN=$$exe ; done


.PHONY: yaml
yaml: check
	set -xe && \
	cd $(THIS_DIR) && \
	  $(FUZZBIN) \
	  $(OPTS_YAML_) \
	  $(OPTS_) \
	  -artifact_prefix=$(FAIL_DIR_YAML) \
	  $(CORPUS_YAML)


.PHONY: json
json: check
	set -xe && \
	cd $(THIS_DIR) && \
	  $(FUZZBIN) \
	  $(OPTS_JSON_) \
	  $(OPTS_) \
	  -artifact_prefix=$(FAIL_DIR_JSON) \
	  $(CORPUS_JSON)


.PHONY: stop
stop:
	killall -SIGUSR1 $(FUZZBIN)


.PHONY: check
check:
	if [ ! -f "$(FUZZBIN)" ] ; then \
	  echo "FUZZBIN not found: $(FUZZBIN)" ; \
	  exit 1 ; \
	fi
	if [ ! -x "$(FUZZBIN)" ] ; then \
	  echo "FUZZBIN not executable: $(FUZZBIN)" ; \
	  exit 1 ; \
	fi
