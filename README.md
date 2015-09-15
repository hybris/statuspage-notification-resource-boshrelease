BOSH release for statuspage-notification-resource
=======================

Background
----------

### What is statuspage-notification-resource?

Send notifications to your [statuspage](https://www.statuspage.io) account from Concourse:

Usage
-----

To use this bosh release, first upload it to the BOSH/bosh-lite that is running Concourse:

```
bosh upload release https://github.com/hybris/statuspage-notification-resource-boshrelease
```

Next, update your Concourse deployment manifest to add the resource.

Add the `statuspage-notification-resource` release to the list:

```yaml
releases:
  - name: concourse
    version: latest
  - name: garden-linux
    version: latest
  - name: statuspage-notification-resource
    version: latest
```

Into the `worker` job, add the `{release: statuspage-notification-resource, name: install}` job template that will install the package:

```yaml
jobs:
- name: worker
  templates:
    ...
    - {release: statuspage-notification-resource, name: install}
```

The final change is to explicitly list all the resource types (they are implicit) and add the `statuspage-notification-resource` package to the list:

```yaml
jobs:
- name: worker
  ...
  properties:
    groundcrew:
      resource_types:
      ...
      - type: statuspage-notification
        image: /var/vcap/packages/statuspage-notification-resource
```

Note that it is the latter two lines that are specific to this BOSH release:

```yaml
- type: statuspage-notification
  image: /var/vcap/packages/statuspage-notification-resource
```

The former lines should be obtained from the Concourse BOSH release, not the documentation above which might be out of date. Use https://github.com/concourse/concourse/blob/master/jobs/groundcrew/spec#L69-L96

And `bosh deploy` your Concourse manifest.

Usage
-----

An example mini-pipeline that would update the status:

```yaml
---
jobs:
- name: hello-world
  plan:
    - task: say-hello
      config:
        platform: linux
        image: "docker:///busybox"
        run:
          path: echo
          args: ["Hello, world!"]
      on_success:
        put: statuspage-update
        params:
          page: <PAGE_ID>
          component: <COMPONENT_ID>
          status: "operational"
      on_failure:
        put: statuspage-update
        params:
          page: <PAGE_ID>
          component: <COMPONENT_ID>
          status: "major_outage"

resources:
- name: statuspage-update
  type: statuspage-notification
  source:
    url: https://api.statuspage.io/v1/pages
    oauth: <OAuth>
```

Setup pipeline in Concourse
---------------------------

```
fly -t concourse1 c -c pipeline.yml --vars-from credentials.yml statuspage-notification-resource-boshrelease
```
