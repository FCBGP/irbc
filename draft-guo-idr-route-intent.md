---
title: "Connection Role Commitment (CRC) for Inter-Domain Routing"
abbrev: "CRC-Intent"
category: info

docname: draft-guo-idr-route-intent-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Inter-Domain Routing"
keyword:
venue:
  group: "Inter-Domain Routing"
  type: "Working Group"
  mail: "idr@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/idr/"
  github: "FCBGP/idr-intent"
  latest: "https://FCBGP.github.io/idr-intent/draft-guo-idr-route-intent.html"

pi:    # can use array (if all yes) or hash here
toc: yes
sortrefs: yes  # defaults to yes
symrefs: yes

author:
-
    name: Yangfei Guo
    org: Zhongguancun Laboratory
    city: Beijing
    country: China
    email: guoyangfei@zgclab.edu.cn

normative:

informative:

...

--- abstract

The Border Gateway Protocol (BGP) provides the primary mechanism for exchanging reachability information between Autonomous Systems (ASes). Existing routing security mechanisms, including the Resource Public Key Infrastructure (RPKI), Route Origin Authorization (ROA), ASPA, and BGPsec, improve the ability to validate different aspects of routing information, such as route origin authorization, autonomous-system relationships, and the authenticity of BGP propagation paths.

However, these mechanisms do not provide a general mechanism for expressing and validating the intended network connection functions or routing behaviors of an AS. In particular, authorization information and observed routing behavior are largely represented and validated independently. As a result, a gap remains between the behavior that an AS is authorized to perform or advertise and the behavior that the operator actually expects the network to provide.

This document introduces the concept of a Connection Role Commitment (CRC), which provides a structured representation of an operator-declared connection role and associated routing behavior constraints. CRC is intended to complement existing routing security mechanisms by providing a declarative basis for expressing expected routing behavior and enabling its subsequent verification against observed BGP behavior.

This document primarily describes the problem statement and the gap in existing inter-domain routing mechanisms. Protocol encoding, distribution, and verification mechanisms are outside the scope of this initial version.



--- middle

# Introduction {#intro}


BGP was designed primarily as a distributed protocol for exchanging reachability information between Autonomous Systems. Routing decisions are consequently influenced by local policy, inter-domain relationships, and protocol attributes. The resulting routing behavior is determined by the interaction of policies configured independently by different network operators.

As the Internet has evolved, routing security mechanisms have been developed to provide stronger assurances about different properties of BGP routing information.

RPKI-based Route Origin Validation (ROV) allows a relying party to determine whether an AS is authorized to originate a particular IP prefix. ASPA provides mechanisms for validating certain properties of the AS_PATH based on provider authorization relationships. BGPsec provides cryptographic protection for the propagation path of BGPsec UPDATE messages.

These mechanisms address important security properties, but they operate primarily on specific properties of routing information. They do not provide a general representation of the network connection function that an operator expects to realize through BGP.

For example, an operator may expect that:

* a customer prefix is originated only by a particular AS;
* traffic toward a destination should normally traverse a designated provider or set of providers;
* a route should not be propagated through a particular class of neighboring ASes;
* a peer relationship should be used for a particular category of reachability;
* a backup path should be used only when a preferred path is unavailable;
* a route should be advertised to one neighbor but not propagated beyond another neighbor; or
* an AS should perform a specific routing role within an inter-domain connection.

Some of these requirements can be partially implemented using existing BGP policy mechanisms, communities, local configuration, or routing attributes. However, the expected behavior is generally encoded implicitly in local configuration and operational policy rather than represented as an explicit, portable, and independently verifiable object.

This creates a fundamental gap between **what an operator declares or expects the network to do** and **what the network can demonstrate that it actually does**.

CRC is proposed as a mechanism to address this gap.

The purpose of CRC is not to replace RPKI, ASPA, BGPsec, or existing BGP policy mechanisms. Instead, CRC provides a higher-level declarative representation of connection roles and expected routing behavior that can be used together with these mechanisms.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Terminology

## Connection Function

A network connection function describes the routing role and expected routing behaviors associated with an inter-domain connection.

## Connection Role

A connection role describes the functional role performed by an AS or routing entity within an inter-domain connection.

## Connection Role Commitment (CRC)

A CRC is a structured declaration representing a connection role and the associated expected routing behavior.

## Expected Behavior

Expected behavior is the routing behavior that an operator declares should be performed under a particular connection function.

## Observed Behavior

Observed behavior is the routing behavior that can be derived from BGP messages, routing state, or other routing evidence.

## Routing Evidence

Routing evidence is information that can be used to establish properties of observed routing behavior, including information provided by BGP, RPKI, ASPA, BGPsec, and related mechanisms.

# Problem Statement

## Implicit Routing Functions

Inter-domain routing behavior is commonly specified through local configuration.

A network operator may configure import and export policies, local preference, route filtering, route redistribution, communities, path manipulation, and other mechanisms to implement an intended network connection function.

The resulting configuration determines the behavior of a router, but the intended function is not necessarily represented as an explicit protocol-level object.

Consequently, an external party observing BGP routing information generally cannot determine the complete expected behavior solely from the information carried by BGP.

This results in an **implicit routing function problem**:

> The network connection function intended by an operator is often embedded in distributed local configuration rather than explicitly represented as a machine-readable and verifiable object.

This is particularly problematic when routing behavior crosses administrative boundaries. A remote AS may observe the resulting BGP behavior without having access to the policy that produced that behavior.

## Separation Between Authorization and Expected Behavior

Existing routing security mechanisms provide strong validation for specific properties.

For example, origin validation can determine whether an AS is authorized to originate a prefix. Path validation can provide assurances about properties of the AS_PATH. Cryptographic mechanisms can protect the integrity or authenticity of routing information.

However, authorization does not necessarily describe the complete expected connection function.

An AS may be authorized to originate a prefix while still advertising or propagating that prefix in a manner inconsistent with the expected routing relationship.

Similarly, a path may be cryptographically valid while still violating an operator's intended routing policy.

Therefore, there is a distinction between:

* **authorization** — what an AS is permitted to do;
* **observed behavior** — what the AS actually does; and
* **expected behavior** — what the operator declares that the AS should do.

Existing routing security mechanisms primarily address the first two categories and do not provide a general representation for the third.

## Lack of an Explicit Reference for Behavior Verification

To determine whether an observed routing behavior is correct, a verifier needs a reference describing what behavior is expected.

In current BGP deployments, this reference is frequently represented by local configuration, bilateral agreements, operational documentation, or implicit assumptions about AS relationships.

These representations are generally not available to remote verifiers and are not necessarily expressed in a common machine-readable form.

As a result, a verifier may be able to determine that:

> "This route was originated by an authorized AS."

or:

> "This AS_PATH satisfies a particular path validation property."

but may not be able to determine:

> "This route propagation behavior satisfies the connection function declared by the relevant network operators."

This limits the ability to perform systematic behavior-level verification.

## Distributed Configuration and Cross-Domain Verification

An inter-domain connection involves multiple independently administered ASes.

The routing behavior observed at one AS can be affected by policies configured by other ASes. Consequently, verifying a connection function may require information originating from multiple administrative domains.

Current mechanisms do not provide a common abstraction for declaring such expectations and associating them with the routing behavior subsequently observed in the network.

The absence of such an abstraction creates a distributed verification gap:

> Operators can configure routing behavior locally, but the corresponding expected connection function is not necessarily distributed together with sufficient information to allow other parties to independently verify the resulting behavior.

## Behavior Is More General Than Route Origin and Path Validity

A routing system can exhibit many distinct behaviors even when origin authorization and path validity are both satisfied.

For example, an AS can:

* accept a route;
* reject a route;
* originate a route;
* advertise a route to a neighbor;
* suppress a route;
* propagate a route;
* prepend its ASN;
* alter route attributes;
* select one route over another;
* provide a primary path;
* provide a backup path; or
* apply different behaviors to different classes of routes or neighbors.

These behaviors can be individually valid from the perspective of existing routing security mechanisms while collectively violating an operator's intended connection function.

Therefore, a routing security architecture that only validates individual route properties cannot fully address behavior-level inconsistencies.

## The "Expected-versus-Observed" Gap

The fundamental problem can be summarized as a gap between **expected routing behavior** and **observed routing behavior**.

Let:

* \(E\) denote the connection function expected or declared by an operator;
* \(B\) denote the routing behavior observed in the network; and
* \(V(B,E)\) denote the verification of whether the observed behavior satisfies the expected function.

Existing routing security mechanisms provide mechanisms for validating particular properties of \(B\), such as origin authorization or path authenticity.

However, there is no general standardized representation of \(E\) that allows a verifier to evaluate:

$$
V(B,E)
$$

across a broad class of inter-domain routing behaviors.

CRC is intended to provide this missing declarative reference.

# Gap Analysis

## Origin Authorization Does Not Express Connection Functions

RPKI and ROA-based origin validation establish an authorization relationship between an IP prefix and an originating AS.

This answers a question of the form:

> "Is AS X authorized to originate prefix P?"

It does not, by itself, describe:

> "How should AS X participate in the propagation of prefix P?"

For example, an ROA does not express whether a route should be exported to a particular neighbor, whether the AS should provide primary or backup connectivity, or whether the route should be propagated beyond a particular relationship.

Thus, origin authorization is necessary for routing security but is not sufficient to represent a complete connection function.

**Gap:** no standardized declarative object connects prefix-origin authorization with the broader routing behavior expected from the originating or transit AS.

## Path Validation Does Not Express Expected Routing Behavior

ASPA and BGPsec address different aspects of path validity.

ASPA can be used to validate relationships represented by AS_PATH, while BGPsec provides cryptographic protection for the propagation of BGPsec routes.

These mechanisms primarily answer questions such as:

> "Is this observed path consistent with the relevant authorization or cryptographic evidence?"

They do not necessarily answer:

> "Is this path the path that the operator expects to be used for this connection function?"

A path can therefore be valid according to a path-validation mechanism while still being undesirable or inconsistent with the declared routing function.

**Gap:** path validity does not establish semantic compliance with an operator-declared connection function.

## BGP Policy Mechanisms Are Powerful but Primarily Local

BGP provides extensive policy mechanisms through route maps, prefix lists, communities, local preference, MED, AS_PATH manipulation, filtering, and other attributes.

These mechanisms can implement sophisticated routing behavior.

However, their semantics are generally evaluated within the administrative domain that configures them. A remote party normally observes only the resulting routing behavior rather than the complete policy intent that generated it.

Consequently, BGP policy mechanisms provide an effective **implementation mechanism**, but not necessarily a standardized **declaration and verification mechanism** for the intended connection function.

**Gap:** existing policy mechanisms can implement routing functions without providing a portable representation of the function itself.

## BGP Communities Convey Policy Signals but Do Not Provide a General Commitment Model

BGP communities and related mechanisms allow an AS to attach policy-related information to routes.

They are widely used for operational signaling, traffic engineering, route filtering, and other purposes.

However, communities generally represent policy signals rather than a complete commitment to a connection function. Their interpretation can also depend on bilateral or local policy.

Consequently, communities do not inherently provide:

1. a complete representation of the expected connection function;
2. a standardized binding between the declared function and the responsible AS;
3. cryptographic protection of the declaration; or
4. a general mechanism for comparing the declaration against observed routing behavior.

**Gap:** policy signaling is not equivalent to a verifiable commitment to an expected routing function.

## Expected Behavior Is Not a First-Class Distributed Object

A fundamental difference exists between routing information and routing expectations.

BGP distributes information about reachable destinations and associated routing attributes. Existing routing security mechanisms provide additional authorization or authenticity information.

The expected connection function, however, is generally maintained as configuration or operational knowledge.

This means that the network lacks a common first-class object representing:

> who performs a connection role, what that role is, and what routing behavior is expected from that role.

Without such an object, behavior verification remains dependent on reconstructing expectations from local configuration, bilateral agreements, or external operational knowledge.

**Gap:** expected routing behavior is not represented as a standardized, distributable, and independently verifiable object.

## No General Binding Between Declared Function and Observed Behavior

A routing security system needs to establish a relationship between a declaration and evidence.

Conceptually, the verification process should be able to determine:

$$
\text{Declared Connection Function}
\rightarrow
\text{Expected Behavior}
\rightarrow
\text{Observed Behavior}
$$

Existing mechanisms provide pieces of this chain, but not a general binding across it.

For example:

* ROA provides authorization evidence for route origin;
* ASPA provides authorization information relevant to AS relationships;
* BGPsec provides cryptographic evidence concerning route propagation; and
* BGP UPDATE messages provide observations of routing behavior.

What is missing is a common semantic object that states **what behavior those observations are expected to satisfy**.

**Gap:** there is no general standardized commitment that binds an operator-declared connection function to observable routing behavior.

## Existing Mechanisms Are Property-Oriented Rather Than Function-Oriented

The current routing security architecture can be viewed as a collection of mechanisms that validate individual properties:

| Mechanism        | Primary property                                                            |
| ---------------- | --------------------------------------------------------------------------- |
| ROA / RPKI       | Prefix-origin authorization                                                 |
| ASPA             | Authorized provider/AS relationship information and path-related validation |
| BGPsec           | Cryptographic protection of BGP propagation path                            |
| BGP communities  | Policy signaling                                                            |
| Local BGP policy | Local routing decision and export/import behavior                           |

These mechanisms are valuable and complementary.

However, they do not provide a common abstraction for a higher-level **connection function** that may combine multiple routing behaviors.

A connection function may consist of multiple constraints simultaneously. For example, a function may specify an origin role, permitted propagation relationships, preferred or backup behavior, and export constraints.

Such a function cannot necessarily be represented by a single existing routing security property.

**Gap:** existing mechanisms validate individual properties, whereas connection functions may require validation of a composition of routing behaviors.

## The Resulting Architecture Gap

The gaps above can be summarized as follows:

```text
                 Existing Routing Security
                 -------------------------
                       |
       +---------------+----------------+
       |               |                |
      ROA             ASPA            BGPsec
       |               |                |
 Origin Authorization  Path/Relation    Path Authenticity
       |               |                |
       +---------------+----------------+
                       |
                 Routing Evidence
                       |
                       v
                Observed Behavior


             Missing Abstraction
             -------------------
                       |
                       v
          Declared Connection Function
                       |
                       v
                Expected Behavior
                       |
                       v
            Behavior-level Verification
```

The missing abstraction is not another independent authorization mechanism.

Rather, it is a mechanism for representing the expected connection function and relating that expectation to observable routing behavior and existing security evidence.

CRC is intended to fill this abstraction gap.

# Design Objectives

Based on the problem and gaps above, a CRC mechanism should satisfy the following objectives.

## Explicit Representation

CRC should provide an explicit and structured representation of a network connection function and the routing role associated with that function.

## Declarative Semantics

CRC should describe expected routing behavior independently from the implementation details used by an individual router to realize that behavior.

## Verifiability

The information represented by CRC should be sufficiently precise to allow a verifier to determine whether observed routing behavior satisfies the declared function.

## Composability

A connection function may consist of multiple atomic routing behaviors. CRC should therefore support the composition of atomic behavioral constraints into a higher-level connection function.

## Compatibility With Existing Routing Security

CRC should complement, rather than replace, existing mechanisms such as RPKI/ROA, ASPA, and BGPsec.

Where applicable, CRC should be able to use evidence provided by existing mechanisms as part of behavior verification.

## Separation of Declaration and Evidence

The declaration of an expected connection function and the evidence of observed routing behavior should remain conceptually distinct.

In this model:

* CRC represents the declared or expected connection function;
* BGP routing information provides observations of routing behavior; and
* existing routing security mechanisms provide authorization or authenticity evidence.

This separation allows the expected behavior and observed behavior to be independently evaluated.

## Extensibility

The representation should support currently identifiable routing behaviors while allowing additional behaviors to be introduced without redesigning the entire mechanism.

# Scope and Non-Goals

This document focuses on the problem statement and gap analysis for representing and verifying network connection functions in inter-domain routing.

The following topics are outside the primary scope of this document:

* defining a complete CRC wire encoding;
* defining a new BGP attribute format;
* specifying the complete distribution mechanism for CRC;
* defining a new public-key infrastructure;
* replacing RPKI, ROA, ASPA, or BGPsec;
* defining a new BGP decision process; or
* specifying a particular implementation architecture.

Future work may define how CRC information is generated, distributed, carried, stored, and verified, including its interaction with BGP and existing routing security mechanisms.


# Security Considerations {#security}


A mechanism representing expected routing behavior introduces security considerations concerning authenticity, integrity, freshness, authorization, and replay of declarations.

In particular, a verifier needs to distinguish between:

* an authentic declaration made by the responsible operator;
* an altered declaration;
* an obsolete declaration; and
* a declaration that is authentic but no longer applicable.

These considerations are expected to be addressed by the protocol mechanisms defined in future versions of this work.

CRC itself should not be considered a replacement for existing routing authorization or path-validation mechanisms. Rather, CRC provides an additional semantic layer for evaluating whether observed routing behavior conforms to an explicitly declared connection function.



# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
