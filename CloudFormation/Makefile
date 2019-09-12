BUCKET ?= gresik
BUCKETREGION ?= us-east-1
STACKREGION ?= us-west-2
STACKNAME ?= k8s-h4rd

.PHONY: package
package: validate
	aws cloudformation package \
		--template ./main.yaml \
		--s3-bucket $(BUCKET) \
		--output-template cfn.yaml \
		--region $(BUCKETREGION);

.PHONY: validate
validate:
	ls | grep .yaml | xargs cfn-lint

.PHONY: deploy
deploy: package
	aws cloudformation deploy \
		--template-file cfn.yaml \
		--stack-name $(STACKNAME) \
		--region $(STACKREGION) \
		--capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
		--parameter-overrides $(shell grep -v '^\s*\(#\|$$\)' parameters.conf)

.PHONY: clean
clean:
	rm -f cfn.yaml;