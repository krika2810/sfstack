# Playbook: deploy

Ship via `/sf-deploy`: preflight (clean SHA, blast-radius on destructive changes), validate-first with the right test level, deploy the validated package, evidence bundle (deployment ID, test-run ID, coverage delta), post-deploy smoke, rollback command written before deploying. Production windows and destructive manifests are human-owned decisions - present, don't decide.
