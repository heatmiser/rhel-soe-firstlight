# scenario testing

## Security testing

There are two ways to test the deployment of security scenarios:

- non-invasive inegration tests
- invasive walkthrough tests

### Integration tests

The integration tests verify that the scenario was successfully deployed. Those tests are non-invasive, they are idempotent and do not alter anything in the deployment.

To run the integration tests, the playbook `security_verify.yml` needs to be run against the instructor inventory of a deployed scenario. The SSH key needs to be provided as a parameter, for example:

```
ansible-playbook provisioner/tests/security_verify.yml -i provisioner/test-scenario/instructor_inventory.txt --private-key=provisioner/test-scenario/test-scenario-private.pem
```

### Walkthrough tests

The walkthrough tests try to resemble the basic operations done through the exercises. Playbooks are created and run, configurations are changed. This makes the tests invasive by nature, they are not idempotent and can only be run once.

To run the walkthrough tests, the playbook `security_exercise_21.yml` needs to be run against each student inventory of a deployed scenario. The SSH key must NOT be provided as a prameter, for example:

```
ansible-playbook provisioner/tests/security_exercise_21.yml -i provisioner/test-scenario/student4-instances.txt
```

## RHEL testing

There is currently a deployment test available. Current tests:

- Availabilitiy of all machines
- Right Ansible version installed on control host
- Necessary Python libs are installed on control host to run Tower modules
- Tower itself is installed
- Tower license is configured and valid (login test)
- Tower is working and provides API feedback

The tower password as well as the scenario name must be provided on the command line.

```
ansible-playbook provisioner/tests/rhel_verify.yml -i provisioner/test-scenario/instructor_inventory.txt --private-key=provisioner/test-scenario/test-scenario-private.pem -e tower_password=mypwd -e scenario_name=test-scenario
```

