---
title: filter-logs-by-pod-label
authors:
  - "@alanconway"
reviewers:
  - TBD
approvers:
  - TBD
creation-date: 2020-08-25
status: provisional
see-also:
  - (https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)[Kubernetes Labels and Selectors]
---

# Filter Logs by Pod Label

## Release Signoff Checklist

- [ ] Enhancement is `implementable`
- [ ] Design details are appropriately documented from clear requirements
- [ ] Test plan is defined
- [ ] Graduation criteria for dev preview, tech preview, GA
- [ ] User-facing documentation is created in [openshift-docs](https://github.com/openshift/openshift-docs/)

## Summary

Forward logs form specific pods to a forwarder output. Kubernetes has two ways
to identify pods: namespaces and labels.  The ClusterLogForwarder (CLF) a,-llready
has an input filter for namespaces, this enhancement will add filtering by labels.

## Motivation

Users want to target specific logs for forwarding. In addition to classifying
logs as applicationd, infrastructure or audit, some users will want to further
select application logs by namespace and labels.

### Goals

* Be able to configure the CLF to forward logs selected by label.
* Support _equality based_ and _set based_ selectors as defined in [https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/](Kubernetes Labels and Selectors)
* Support the same expressions as the `--selector` flag of `kubectk` or `oc`

## Proposal

Extend the `clusterlogforwarder.inputs.application` to handle label map and
expression selectors:

```go
type Application struct {
	// Namespaces selects logs from pods in one of the listend namespaces.
	//
	// +optional
	Namespaces []string `json:"namespaces,omitempty"`

	// Labels selects logs from pods with all the given label values.
	//
	// +optional
	Labels map[string]string `json:"labels,omitempty"`

	// Expressions selects logs from pods that match all the selector expressions,
	// in the same format as `kubectl get --selector` or `oc get --selector`
	//
	// +optional
	Expressions string `json:"expressions,omitempty"`
}
```

### User Stories

#### Select logs by label map

As a log administator I want to forward logs using a map selector:


```yaml
apiVersion: "logging.openshift.io/v1"
kind: "ClusterLogForwarder"
metadata:
  name: "instance"
spec:
  pipelines:
    - inputRefs: [ myLogs ]
      outputRefs: [ default  ]

  inputs:
    - name: myLogs
      application:
        labels:
          environment: production
          app: nginx
```

#### Select logs by label expressions

As a log administator I want to select logs by label and namespace.


```yaml
apiVersion: "logging.openshift.io/v1"
kind: "ClusterLogForwarder"
metadata:
  name: "instance"
spec:
  pipelines:
    - inputRefs: [ myLogs ]
      outputRefs: [ default  ]

  inputs:
    - name: myLogs
      application:
        labels: { environment: production }
        namespaces: [ app1,  app2 ]
```

### Implementation Details/Notes/Constraints [optional]

TODO

### Risks and Mitigations

TODO

What are the risks of this proposal and how do we mitigate. Think broadly. For
example, consider both security and how this will impact the larger OKD
ecosystem.

How will security be reviewed and by whom? How will UX be reviewed and by whom?

Consider including folks that also work outside your immediate sub-project.

## Design Details

### Open Questions [optional]

TODO

This is where to call out areas of the design that require closure before deciding
to implement the design.  For instance,
 > 1. This requires exposing previously private resources which contain sensitive
  information.  Can we do this?

## Implementation History

TODO

## Drawbacks

TODO

The idea is to find the best form of an argument why this enhancement should _not_ be implemented.

## Alternatives

TODO

Similar to the `Drawbacks` section the `Alternatives` section is used to
highlight and record other possible approaches to delivering the value proposed
by an enhancement.

