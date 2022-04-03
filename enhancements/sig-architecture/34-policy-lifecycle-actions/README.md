# Policy lifecycle actions

## Release Signoff Checklist

- [ ] Enhancement is `implementable`
- [ ] Design details are appropriately documented from clear requirements
- [ ] Test plan is defined
- [ ] Graduation criteria for dev preview, tech preview, GA
- [ ] User-facing documentation is created in [website](https://github.com/open-cluster-management-io/website/)

## Summary

The proposed work would enhance the Policy APIs to allow more flexible alignment of  managed cluster resources with the lifecycle of the owning policy.

## Motivation

Policy is a useful tool for creating objects and enforcing or monitoring the compliance of multiple managed clusters. However, when a policy is deleted, the correspondent objects on the managed clusters remain orphaned.
The existing Policy API allows user to bring the clusters to the desired state using a different set of policies, or other custom APIs, such as ManagedClusterAction or ManifestWork. However, this incurs a significant complexity for the user workflows.

### User Stories
#### Story 1: Policy deletion.
As a user I want that the child objects created by my policy will be deleted if I delete this policy.
##### __Use case__
As a part of the Gitops workflow, a user created a custom resource on a set of spoke clusters using a MustHave complianceType policy. In the next OCP version, this custom resource is no longer needed, so the user removes this policy from the git repository. The policy has been deleted, but the child objects on the spokes are orphaned. When the user applies her Gitops repository the a mix of upgraded and newly installed clusters of the same version, the result is inconsistent, since the upgraded clusters contain the orphaned object, and the newly installed clusters don't. 


#### Story 2: Policy objects subset deletion
As a user I want that the Policy child objects will be deleted from spokes if I remove them from the policy
##### __Use case__
As a part of the Gitops workflow a user created a list of custom resources on a set of spoke clusters using a single MustHave complianceType policy. In the next OCP version, one of the custom resources is no longer needed, so the user deletes this object from the policy-templates list in the git repository and synchronizes it to the hub. The policy on the hub reflects the repository status, but the object remains orphaned on the spokes. Same inconsistent result as in Story 1.


#### Story 3: Control which objects are orphaned upon the policy deletion
As a user I want to control which policy controlled objects will be orphaned upon the policy deletion, and which won't.
##### __Use case__
A user deletes a policy that contains a MachineConfig manifest and other custom resources. The user wants to keep the MachineConfig orphaned on the spokes to avoid a reboot, but delete all other objects controlled by the policy.

#### Story 4: Control which objects are orphaned upon placement condition change
As a user I want to control which policy controlled objects will be orphaned upon the policy placement condition change, and which won't.
##### __Use case__
A user has a set of spoke clusters with a policy applied and compliant. The user wants to remove a subset of objects from a single cluster without affecting others.
The user modifies a policy by specifying which objects must be deleted upon the placement condition change, and which must remain. After that the user deletes the spoke label that satisfied the relevant PlacementRule clusterSelector. Expected result - the policy does not apply to the spoke anymore, selected objects have been orphaned and others - deleted.


### Open issues
the user wants to switch from policy X to policy Y which are really similar. Is there a way to do this without deleting?
The policy adds just a few fields to an existing CR -- delete the whole CR, or just revert to the pre-managed state? What if the user updates others of those fields?


Unchanged template below this line:
----------

### Goals

List the specific goals of the proposal. How will we know that this has succeeded?

### Non-Goals

What is out of scope for this proposal? Listing non-goals helps to focus discussion
and make progress.

## Proposal

This is where we get down to the nitty gritty of what the proposal actually is.

### User Stories

Detail the things that people will be able to do if this is implemented.
Include as much detail as possible so that people can understand the "how" of
the system. The goal here is to make this feel real for users without getting
bogged down.

#### Story 1

#### Story 2

### Implementation Details/Notes/Constraints [optional]

What are the caveats to the implementation? What are some important details that
didn't come across above. Go in to as much detail as necessary here. This might
be a good place to talk about core concepts and how they relate.

### Risks and Mitigation

What are the risks of this proposal and how do we mitigate. Think broadly. For
example, consider both security and how this will impact the larger OKD
ecosystem.

How will security be reviewed and by whom? How will UX be reviewed and by whom?

Consider including folks that also work outside your immediate sub-project.

## Design Details

### Open Questions [optional]

This is where to call out areas of the design that require closure before deciding
to implement the design.  For instance, 
 > 1. This requires exposing previously private resources which contain sensitive
  information.  Can we do this? 

### Test Plan

**Note:** *Section not required until targeted at a release.*

Consider the following in developing a test plan for this enhancement:
- Will there be e2e and integration tests, in addition to unit tests?
- How will it be tested in isolation vs with other components?

No need to outline all of the test cases, just the general strategy. Anything
that would count as tricky in the implementation and anything particularly
challenging to test should be called out.

All code is expected to have adequate tests (eventually with coverage
expectations). 

All code is expected to have sufficient e2e tests.

### Graduation Criteria

**Note:** *Section not required until targeted at a release.*

Define graduation milestones.

These may be defined in terms of API maturity, or as something else. Initial proposal
should keep this high-level with a focus on what signals will be looked at to
determine graduation.

Consider the following in developing the graduation criteria for this
enhancement:

- [Maturity levels][maturity-levels]
- [Deprecation policy][deprecation-policy]

Clearly define what graduation means by either linking to the [API doc definition](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-versioning),
or by redefining what graduation means.

In general, we try to use the same stages (alpha, beta, stable), regardless how the functionality is accessed.

[maturity-levels]: https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#alpha-beta-and-stable-versions
[deprecation-policy]: https://kubernetes.io/docs/reference/using-api/deprecation-policy/

### Upgrade / Downgrade Strategy

If applicable, how will the component be upgraded and downgraded? Make sure this
is in the test plan.

Consider the following in developing an upgrade/downgrade strategy for this
enhancement:
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade in order to keep previous behavior?
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade in order to make use of the enhancement?

Upgrade expectations:
- Each component should remain available for user requests and
  workloads during upgrades. Any exception to this should be
  identified and discussed here.
- Micro version upgrades - users should be able to skip forward versions within a
  minor release stream without being required to pass through intermediate
  versions - i.e. `x.y.N->x.y.N+2` should work without requiring `x.y.N->x.y.N+1`
  as an intermediate step.
- Minor version upgrades - you only need to support `x.N->x.N+1` upgrade
  steps. So, for example, it is acceptable to require a user running 0.1.0 to
  upgrade to 0.3.0 with a `0.1.0->0.2.0` step followed by a `0.2.0->0.3.0` step.
- While an upgrade is in progress, new component versions should
  continue to operate correctly in concert with older component
  versions (aka "version skew"). For example, if an agent operator on a managed cluster 
  is rolling out a new agent, the old and new agent in different managed clusters
  must continue to work correctly with old or new version of the hub while hub and agents
  remain in this partially upgraded state for some time.

Downgrade expectations:
- If an `N->N+1` upgrade fails mid-way through, or if the `N+1` agent is
  misbehaving, it should be possible for the user to rollback to `N`. It is
  acceptable to require some documented manual steps in order to fully restore
  the downgraded agent to its previous state. Examples of acceptable steps
  include:
  - Deleting any resources added by the new version of the agent operator in the
    managed cluster. The agent operator does not currently delete resources that 
    no longer exist in the target version.

### Version Skew Strategy

How will the component handle version skew with other components?
What are the guarantees? Make sure this is in the test plan.

Consider the following in developing a version skew strategy for this
enhancement:
- During an upgrade, we will always have skew among components, how will this impact your work?
- Does this enhancement involve coordinating behavior in the hub and
  in the klusterlet? How does an n-2 klusterlet without this feature available behave
  when this feature is used?
- Will any other components on the managed cluster change? For example, changes to infrastructure
  configuration of the managed cluster may be needed before the updating klusterlet or other
  agent components.

## Implementation History

Major milestones in the life cycle of a proposal should be tracked in `Implementation
History`.

## Drawbacks

The idea is to find the best form of an argument why this enhancement should _not_ be implemented.

## Alternatives

Similar to the `Drawbacks` section the `Alternatives` section is used to
highlight and record other possible approaches to delivering the value proposed
by an enhancement.

## Infrastructure Needed [optional]

Use this section if you need things from the project. Examples include a new
subproject, repos requested, github details, and/or testing infrastructure.

Listing these here allows the community to get the process for these resources
started right away.