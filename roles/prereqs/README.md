# prereqs

Ensures a Python interpreter exists on the target so that Ansible's modules can
run, using `raw` because no module can run before it.

Runs as the first play of most playbooks, with `gather_facts: false`.

On every currently supported target this is a no-op: modern Ubuntu, RHEL and
Amazon Linux all ship python3. The role exists for older hosts, and reports
`ok` rather than `changed` because `raw` cannot tell whether it did anything.

Failures are ignored deliberately - if the interpreter genuinely is missing,
the first real task will say so far more clearly than this one can.
