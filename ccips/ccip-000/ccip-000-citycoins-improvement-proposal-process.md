# CCIP-000

## Preamble

| CCIP Number   | 000                                    |
| ------------- | -------------------------------------- |
| Title         | CityCoins Improvement Proposal Process |
| Author(s)     | Jason Schrader jason@joinfreehold.com  |
| Consideration | Governance                             |
| Type          | Meta                                   |
| Status        | Draft                                  |
| Created       | 2022-03-15                             |
| License       | BSD-2-Clause                           |

## Introduction

A CityCoins Improvement Proposal (CCIP) is a design document that provides information to the greater CityCoins ecosystem's participants concerning the design of the CityCoins Protocol and its ongoing operation.

Each CCIP shall provide a clear description of features, processes, and/or standards for the CityCoins Protocol and its ongoing operation, with sufficient details provided such that a reasonable practitioner may use the document to create an independent but compatible implementation of the proposed improvement.

CCIPs are the canonical medium by which new features are proposed and described, and by which input from the CityCoins ecosystem participants is collected. The CCIP Ratification Process is also described in this document, and provides the means by which CCIPs may be proposed, vetted, edited, accepted, rejected, implemented, and finally incorporated into the CityCoins Protocol's design, governance, and operational procedures.

The set of CCIPs that have been ratified shall sufficiently describe the design, governance, and operationalization of the CityCoins Protocol, as well as the means by which future changes to its official design, implementation, operation, and governance may be incorporated.

The goals of this process are to ensure that anyone may submit a CCIP in good faith, that each CCIP will receive fair and speedy good-faith consideration by other people with the relevant expertise, and that any discussions and decision-making on each CCIP's ratification shall happen in public. To achieve these ends, this document proposes a standard way of presenting a CityCoins Improvement Proposal (CCIP), and a standard way of ratifying one.

This document uses the word “users” to refer specifically to people who participate in the greater CityCoins ecosystem. This includes, but is not limited to, people who mine CityCoins, people who contribute code, people who develop applications that rely on the CityCoins protocol, people who use such applications, people involved in the project governance, and people involved in operating software deployments.

## License and Copyright

This CCIP is made available under the terms of the BSD-2-Clause license, available at https://opensource.org/licenses/BSD-2-Clause.

A large amount of the CCIP design follows the [Stacks SIP process](https://github.com/stacksgov/sips) for the Stacks blockchain.

## Specification

Each CCIP shall adhere to the same general formatting and shall be ratified through the processes described by this document.

### CCIP Format

All CCIPs shall be formatted as markdown files. Each section shall be annotated as a 2nd-level header (e.g. `##`). Subsections may be added with lower-level headers.

Each CCIP shall contain the following sections, in the given order:

**Preamble**

This section shall provide fields useful for categorizing the CCIP.

The required fields in all cases shall be:

- _CCIP Number._ Each CCIP receives a unique number once it has been accepted for consideration for ratification (see below). This number is assigned to a CCIP; its author does not provide it.
- _Title._ A concise description of the CCIP, no more than 20 words long.
- _Author._ A list of names and email addresses of the CCIP's author(s).
- _Consideration._ What class of CCIP this is ([defined below](#ccip-considerations)).
- _Type._ The CCIP track for consideration ([defined below](#ccip-types)).
- _Status._ This CCIP's point in the CCIP workflow ([defined below](#ccip-workflow)).
- _Created._ The ISO 8601 date when this CCIP was created.
- _License._ The content license for the CCIP (see below for permitted licenses).
- _Sign-off._ The list of relevant persons and their titles who have worked to ratify the CCIP. This field is not filled in entirely until ratification, but is incrementally filled in as the CCIP progresses through the ratification process.

Additional CCIP fields, which are sometimes required, include:

- _Requires._ A list of CCIPs that must be implemented prior to this CCIP.
- _Replaces._ A list of CCIPs that this CCIP replaces.
- _Superceded-By._ A list of CCIPs that replace this CCIP.

**Introduction**

This section shall provide a high-level summary of the problem(s) that this CCIP proposes to solve, as well as a high-level description of how the proposal solves them. This section shall emphasize its novel contributions, and briefly describe how they address the problem(s). Any motivational arguments and example problems and solutions belong in this section.

**Specification**

This section shall provide the detailed technical specification. It may include code snippits, diagrams, performance evaluations, and other supplemental data to justify particular design decisions. However, a copy of all external supplemental data (such as links to research papers) must be included with the CCIP, and must be made available under an approved copyright license.

**Related Work**

This section shall summarize alternative solutions that address the same or similar problems, and briefly describe why they are not adequate solutions. This section may reference alternative solutions in other blockchain projects, in research papers from academia and industry, other open-source projects, and so on. This section must be accompanied by a bibliography of sufficient detail such that someone reading the CCIP can find and evaluate the related works.

**Backwards Compatibility**

This section shall address any backwards-incompatiblity concerns that may arise with the implementation of this CCIP, as well as describe (or reference) technical mitigations for breaking changes. This section may be left blank for non-technical CCIPs.

**Activation**

This section shall describe the timeline, falsifiable criteria, and process for activating the CCIP once it is ratified. This applies to both technical and non-technical CCIPs. This section is used to unambiguously determine whether or not the CCIP has been accepted by the CityCoins users once it has been submitted for ratification (see below).

**Reference Implementations**

This section shall include one or more references to one or more production-quality implementations of the CCIP, if applicable. This section is only informative — the CCIP ratification process is independent of any engineering processes (or other processes) that would be followed to produce implementations. If a particular implementation process is desired, then a detailed description of the process must be included in the Activation section. This section may be updated after a CCIP is ratified in order to include an up-to-date listing of any implementations or embodiments of the CCIP.

Additional sections may be included as appropriate.

### Supplemental Materials

CCIPs may include any supplemental materials as appropriate (within reason), but all materials must have an open format unencumbered by legal restrictions.

For example, a LibreOffice `.odp` slide-deck file may be submitted as supplementary material, but not a Keynote `.key` file.

When submitting the CCIP, supplementary materials must be present within the same directory, and must be named as `CCIP-XXXX-YYY.ext`, where:

- `XXXX` is the CCIP number
- `YYY` is the serial number of the file, starting with 1
- `.ext` is the file extension

## CCIP Considerations

A CCIP's consideration determines the particular steps needed to ratify the CCIP and incorporate it into the CityCoins Protocol. Different CCIP considerations have different criteria for ratification.

A CCIP can have more than one consideration, since a CCIP may need to be vetted by different users with different domains of expertise.

- _Technical._ The CCIP is technical in nature, must be vetted by users with the relevant technical expertise, and must adhere to [RFC 2119](http://www.faqs.org/rfcs/rfc2119.html).
- _Economic._ The CCIP concerns the protocol's token economics.
- _Governance._ The CCIP concerns the governance of the CityCoins Protocol, including the CCIP process. This includes amendments to the CCIP Ratification Process, as well as structural considerations such as the creation (or removal) of various committees, editorial bodies, and formally recognized special interest groups.
- _Ethics._ This CCIP concerns the behaviors of those involved in the CCIP Ratification Process that can affect its widespread adoption. Such CCIPs describe what behaviors shall be deemed acceptable, and which behaviors shall be considered harmful to this end (including any remediation or loss of privileges that misbehavior may entail). CCIPs that propose formalizations of ethics like codes of conduct, procedures for conflict resolution, criteria for involvement in governance, and so on would belong in this CCIP consideration.
- _Diversity._ This CCIP concerns proposals to grow the set of users, with an emphasis on including users who are traditionally not involved with open-source software projects. CCIPs that are concerned with evangelism, advertising, outreach, and so on must have this consideration.

New considerations may be created via the ratification of a Meta-type CCIP under the governance consideration.

## CCIP Types

The types of CCIPs are as follows:

- _Upgrade._ This CCIP type means that all CityCoins would need to adopt this CCIP to remain compatible with one another.
- _Standard._ This CCIP type means that the proposed change affects one or more implementations, but does not affect all CityCoins.
- _Meta._ This CCIP type means that the proposal concerns the CCIP ratification process. Such a CCIP is a proposal to change the way CCIPs are handled.
- _Informational._ This is a CCIP type that provides useful information, but does not require any action to be taken on the part of any user.

New types of CCIPs may be created with the ratification of a Meta-type CCIP under the governance consideration (see above). CCIP types may not be removed.

## CCIP Workflow

As a CCIP is considered for ratification, it passes through multiple statuses as determined by the community.

A CCIP may have exactly one of the following statuses at any given time:

- _Draft._ The CCIP is still being prepared for formal submission. It does not yet have a CCIP number.
- _Accepted._ The CCIP text is sufficiently complete that it constitutes a well-formed CCIP, and is of sufficient quality that it may be considered for ratification. A CCIP receives a CCIP number when it is moved into the Accepted state.
- _Recommended._ The people responsible for vetting the CCIPs under the consideration(s) in which they have expertise have agreed that this CCIP should be implemented. A CCIP must be Accepted before it can be Recommended.
- _Activation-In-Progress._ The CCIP has been tentatively approved for ratification. However, not all of the criteria for ratification have been met according to the CCIP’s Activation section. For example, the Activation section might require miners to vote on activating the CCIPs’ implementations, which would occur after the CCIP has been transferred into Activation-In-Progress status but before it is transferred to Ratified status.
- _Ratified._ The CCIP has been activated according to the procedures described in its Activation section. Once ratified, a CCIP remains ratified in perpetuity, but a subsequent CCIP may supersede it. If the CCIP is an Upgrade-type CCIP, and then all CityCoins implementations must implement it. A CCIP must be Recommended before it can be Ratified. Moving a CCIP into this state may be done retroactively, once the CCIP has been activated according to the terms in its Activation section.
- _Rejected._ The CCIP does not meet at least one of the criteria for ratification in its current form. A CCIP can become Rejected from any state, except Ratified. If a CCIP is moved to the Rejected state, then it may be re-submitted as a Draft.
- _Obsolete._ The CCIP is deprecated, but its candidacy for ratification has not been officially withdrawn (e.g. it may warrant further discussion). An Obsolete CCIP may not be ratified, and will ultimately be Withdrawn.
- _Replaced._ The CCIP has been superseded by a different CCIP. Its preamble must have a Superseded-By field. A Replaced CCIP may not be ratified, nor may it be re-submitted as a Draft-status CCIP. It must be transitioned to a Withdrawn state once the CCIP(s) that replace it have been processed.
- _Withdrawn._ The CCIP's authors have ceased working on the CCIP. A Withdrawn CCIP may not be ratified, and may not be re-submitted as a Draft. It must be re-assigned a CCIP number if taken up again.

The act of ratifying a CCIP is the act of transitioning it to the Ratified status -- that is, moving it from Draft to Accepted, from Accepted to Recommended, and Recommended to Activation-In-Progress, and from Activation-In-Progress to Ratified, all without the CCIP being transitioned to Rejected, Obsolete, Replaced, or Withdrawn status. A CCIP's current status is recorded in its Status field in its preamble.

## CCIP Workflow Diagram

```none
------------------
|     Draft      |  <-------------------------. Revise and resubmit
------------------                            |
        |                             --------------------
Submit to CCIP Editor ------------->  |     Rejected     |
        |                             --------------------
        |                                     ^
        V                                     |
------------------                            |
|   Accepted     | -------------------------/ | /--------------------------------.
------------------                            |                                  |
        |                             --------------------                       |
Review by Community --------------->  |     Rejected     |                       |
        |                             --------------------                       |
        |                                     ^                                  |
        V                                     |                                  |
-------------------------                     |                                  |
|      Recommended      | =-----------------/ | /------------------------------->|
-------------------------                     |                                  |
        |                              --------------------                      |
Vote with Smart Contract ---------->   |    Rejected      |                      |
        |                              --------------------                      |
        |                                     ^                                  |
        V                                     |                                  |
--------------------------                    |                                  |
| Activation-in-Progress | -----------------/ | /------------------------------->|
--------------------------                    |                                  |
        |                            ---------------------                       |
All activation  ------------------>  |     Rejected      |                       |
criteria are met        |            ---------------------   ------------------  |
        |               |----------------------------------> |    Obsolete    |  |
        V               |     ---------------------          ------------------  |
------------------     *--->  |     Replaced      | --------------->|<-----------*
|   Ratified     |            ---------------------                 |
------------------                                                  V
                                                            -------------------
                                                            |    Withdrawn    |
                                                            -------------------
```

When a CCIP is transitioned to Rejected, it is not deleted, but is preserved in the CCIP repository so that it can be referenced as related or prior work by other CCIPs. Once a CCIP is Rejected, it may be re-submitted as a Draft at a later date. CCIP Editors may decide how often to re-consider rejected CCIPs as an anti-spam measure, but the Steering Committee and Consideration Advisory Boards may opt to independently re-consider rejected CCIPs at their own discretion.

Once a CCIP has been moved to Ratified status, the only changes that may be made to it are fixing errata and adding supplementary materials. Substantial changes to the CCIP's body should be done as a separate CCIP.

## Public Venues for Conducting Business

The canonical set of CCIPs in all state shall be recorded in the same medium that the canonical copy of this CCIP is. Right now, this is in the Github repository https://github.com/citycoins/governance, but may be changed before this CCIP is ratified. New CCIPs, edits to CCIPs, comments on CCIPs, and so on shall be conducted through Github's facilities for the time being.

### Github-specific Considerations

All CCIPs shall be submitted as pull requests, and all CCIP edits (including status updates) shall be submitted as pull requests. The community shall be responsible for merging pull requests to the main branch.

## CCIP Copyright & Licensing

Each CCIP must identify at least one acceptable license in its preamble. Source code in the CCIP can be licensed differently than the text. CCIPs whose reference implementation(s) touch existing reference implementation(s) must use the same license as the existing implementation(s) in order to be considered. Below is a list of recommended licenses.

- BSD-2-Clause: OSI-approved BSD 2-clause license
- BSD-3-Clause: OSI-approved BSD 3-clause license
- CC0-1.0: Creative Commons CC0 1.0 Universal
- GNU-All-Permissive: GNU All-Permissive License
- GPL-2.0+: GNU General Public License (GPL), version 2 or newer
- LGPL-2.1+: GNU Lesser General Public License (LGPL), version 2.1 or newer

## Related Work

The governance process proposed in this CCIP is inspired by the Stacks Blockchain SIP process[^1], which drew inspiration from the Python PEP process[^2], the Bitcoin BIP2 process[^3], the Ethereum Improvement Proposal[^4] processes, the Zcash governance process[^5], and the Debian GNU/Linux distribution governance process[^6].

[^1]: https://github.com/stacksgov/sips
[^2]: https://www.python.org/dev/peps/pep-0001/
[^3]: https://github.com/bitcoin/bips/blob/master/bip-0002.mediawiki
[^4]: https://eips.ethereum.org/
[^5]: https://www.zfnd.org/governance/
[^6]: https://debian-handbook.info/browse/stable/sect.debian-internals.html
