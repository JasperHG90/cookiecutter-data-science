.PHONY: clean data lint requirements sync_data_to_s3 sync_data_from_s3

#################################################################################
# GLOBALS                                                                       #
#################################################################################

PROJECT_DIR := $(shell dirname $(realpath $(lastword $(MAKEFILE_LIST))))
PROJECT_NAME = {{ cookiecutter.repo_name }}
{% if cookiecutter.cloud_vendor != 'none' %}
{% if cookiecutter.cloud_vendor == 'AWS' -%}
PROFILE = {{ cookiecutter.project_profile_or_id }}
CLOUD_VENDOR = AWS
{% elif cookiecutter.cloud_vendor == 'GCP' -%}
CLOUD_VENDOR = GCP
PROJECT_ID = {{ cookiecutter.project_profile_or_id }}
{% endif %}
DATA_BUCKET = {{ cookiecutter.data_bucket }}
{% endif %}

#################################################################################
# COMMANDS                                                                      #
#################################################################################

## Upgrade pip
update-pip:
	poetry run python -m pip install -U pip

## Install python dependencies
setup: update-pip
	poetry install

## Install common DS libraries (pandas, numpy, dask, matplotlib, dask-ml, scipy, jupyter, numba, mlflow)
install-ds-libs: setup
	poetry add pandas numpy scipy matplotlib dask dask-ml jupyter ipykernel numba mlflow

## Make Dataset
data: setup
	poetry run python {{ cookiecutter.module_name }}/data/make_dataset.py data/raw data/processed

## Delete all compiled Python files
clean:
	find . -type f -name "*.py[co]" -delete
	find . -type d -name "__pycache__" -delete

## Lint using flake8
lint: setup
	poetry run flake8 {{ cookiecutter.module_name }}

## Run python tests
test: setup
	poetry run pytest $(PROJECT_DIR)

## Set up pre-commit
setup-pre-commit:
	poetry run pre-commit install

{% if cookiecutter.cloud_vendor == 'GCP' -%}
## Set google project id
set-gcp-proj-id:
	gcloud config set project $(PROJECT_ID)
{% endif %}

{% if cookiecutter.cloud_vendor != 'none' %}
## Download Data from storage system
sync_data_down:
	{% if cookiecutter.cloud_vendor == 'AWS' -%}
	aws s3 sync s3://$(DATA_BUCKET)/data/\
		data/ --profile $(PROFILE)
	{% elif cookiecutter.cloud_vendor == 'GCP' -%}
	gsutil -m rsync -r gs://$(DATA_BUCKET)/data/ data/
	{% endif %}

## Upload Data to storage system
sync_data_up:
	{% if cookiecutter.cloud_vendor == 'AWS' -%}
	aws s3 sync s3://$(DATA_BUCKET)/data/ data/\
		--profile $(PROFILE)
	{% elif cookiecutter.cloud_vendor == 'GCP' -%}
	gsutil -m rsync -r data/ gs://$(DATA_BUCKET)/data/
	{% endif %}
{% endif %}

#################################################################################
# PROJECT RULES                                                                 #
#################################################################################



#################################################################################
# Self Documenting Commands                                                     #
#################################################################################

.DEFAULT_GOAL := help

# Inspired by <http://marmelab.com/blog/2016/02/29/auto-documented-makefile.html>
# sed script explained:
# /^##/:
# 	* save line in hold space
# 	* purge line
# 	* Loop:
# 		* append newline + line to hold space
# 		* go to next line
# 		* if line starts with doc comment, strip comment character off and loop
# 	* remove target prerequisites
# 	* append hold space (+ newline) to line
# 	* replace newline plus comments by `---`
# 	* print line
# Separate expressions are necessary because labels cannot be delimited by
# semicolon; see <http://stackoverflow.com/a/11799865/1968>
.PHONY: help
help:
	@echo "$$(tput bold)Available rules:$$(tput sgr0)"
	@echo
	@sed -n -e "/^## / { \
		h; \
		s/.*//; \
		:doc" \
		-e "H; \
		n; \
		s/^## //; \
		t doc" \
		-e "s/:.*//; \
		G; \
		s/\\n## /---/; \
		s/\\n/ /g; \
		p; \
	}" ${MAKEFILE_LIST} \
	| LC_ALL='C' sort --ignore-case \
	| awk -F '---' \
		-v ncol=$$(tput cols) \
		-v indent=19 \
		-v col_on="$$(tput setaf 6)" \
		-v col_off="$$(tput sgr0)" \
	'{ \
		printf "%s%*s%s ", col_on, -indent, $$1, col_off; \
		n = split($$2, words, " "); \
		line_length = ncol - indent; \
		for (i = 1; i <= n; i++) { \
			line_length -= length(words[i]) + 1; \
			if (line_length <= 0) { \
				line_length = ncol - indent - length(words[i]) - 1; \
				printf "\n%*s ", -indent, " "; \
			} \
			printf "%s ", words[i]; \
		} \
		printf "\n"; \
	}' \
	| more $(shell test $(shell uname) = Darwin && echo '--no-init --raw-control-chars')
