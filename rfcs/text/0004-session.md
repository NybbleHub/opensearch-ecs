# 0004: Session
<!--^ The ECS team will assign a unique, contiguous RFC number upon merging the initial stage of this RFC, taking care not to conflict with other RFCs.-->

- Stage: **0 (strawperson)** <!-- Update to reflect target stage -->
- Date: 7/30/2020 <!-- Update to reflect date of most recent stage advancement -->

<!--
Stage 0: Provide a high level summary of the premise of these changes. Briefly describe the nature, purpose, and impact of the changes. ~2-5 sentences.
-->

## Fields
This RFC calls for the addition of session fields to describe events related to various types of "sessions" reported by appliances, security devices, systems, management portals, applications, etc.

| Field | Description |
| ----- | ----------- |
|session.kind:            | local, remote, network
|session.authorization:   | user, admin, service
|session.type:            | system, virtual, application, wired, wireless, vpn
|session.name             | locally relevant name if available (e.g. HQ Client VPN, Win19-VDI, FIN-EXCEL-vApp)
|session.id               | session id provided by server or custom fingerprint



## Fields (yaml)
```yaml
---
- name: session
  title: Session
  group: 2
  short: User, admin, application, network, or service sessions
  description: |-
    These fields are used to track an entity's interaction with various assets, services, and applications in an enterprise.  Sessions will typically include a start event, often a login / authorization event performed locally or via network based mechanisms, and an end event indicating a logoff or session termination.  Related events during the scope of the session will typically be associated via tuples of user, source and or destination ip/port, or cookies.

    When available, event start/end or duration fields should be populated, as well as iam, user, network, host, observer, process, source, destination, client, and server fields as appropriate to describe the specifics of the interaction.

  type: group

  fields:

    - name: kind
      level: extended
      type: keyword
      short: Kind of session
      description: > Session kind can be local (console, on the keyboard), remote (ssh, vdi, web, ftp), or network (802.1x, wpa, NAC)
      Additional fields will be dependent on the specifics of the session reported.

      example: network

    - name: authorization
      level: extended
      type: keyword
      description: Authorization scope of the session. Initial values will include general user level access (e.g. user vdi/vda, vpn, or web sessions, network access, etc), administrative sessions (root, VMWare Host access, router cli, etc.) or service (network to network VPN, non-user verified services sessions e.g. micro-service backend architectures).

      example: user

    - name: type
      level: extended
      type: Logical session type
      description: Session type describes the interaction/access provided.  Initial values include system (shell or desktop), virtual (VDI), application (web, ftp, etc.), wired (nac, 802.1x), wireless (wpa/.1x), or vpn (ipsec, ssl, etc). Note that actual aaa mechanism (system, domain, wpa, 802.1x) does not indicate a specific session type.

      example: wireless

    - name: name
      level: extended
      type: Session Name
      description: The name field is meant to contain a locally significant identifier for the session as configured. This could represent a VPN group name, a wireless network name (ssid), a wired network segment, VDI service name, or application identifier.

      example: HQ-Wireless

    - name: id
      level: extended
      type: Session id
      description: The id field is meant to contain a locally significant identifier for the session as provided by the observer or host reporting the session.  If no id is provided this field can remain blank, or a hash function similar to network.community_id can be used to discretely identify sessions from unique values.

      example: 7635344
```
<!--
Stage: 1: Describe at a high level how this change affects fields. Which fieldsets will be impacted? How many fields overall? Are we primarily adding fields, removing fields, or changing existing fields? The goal here is to understand the fundamental technical implications and likely extent of these changes. ~2-5 sentences.
-->

<!--
Stage 2: Include new or updated yml field definitions for all of the essential fields in this draft. While not exhaustive, the fields documented here should be comprehensive enough to deeply evaluate the technical considerations of this change. The goal here is to validate the technical details for all essential fields and to provide a basis for adding experimental field definitions to the schema. Use GitHub code blocks with yml syntax formatting.
-->

<!--
Stage 3: Add or update all remaining field definitions. The list should now be exhaustive. The goal here is to validate the technical details of all remaining fields and to provide a basis for releasing these field definitions as beta in the schema. Use GitHub code blocks with yml syntax formatting.
-->

## Usage

Session fields are used to describe the sesison attributes of:
 - Client VPN Sessions
 - Network to Network VPN Sessions
 - Network Access Sessions (NAC, WPA, EAP, etc.)
 - Local or remote device login sessions (RDP, ICA, xWindows)
 - Administrative sessions on infrastructure devices
 - Administrative sessions on cloud or application management portals
 - Applications sessions (e.g. sql server odbc session, application access session)


## Source data
Source data expectations include:
 - Wireless Lan Controllers
 - Security appliances
 - Network admission control devices
 - Radius / tacacs servers
 - Application server logs

Example 1: Meraki 802.1x Logs (WLC)
* EAP session start)
    * `<134>1 1580551704.928047208 my_AP events type=8021x_eap_success radio='1' vap='2' client_mac='12:34:56:78:9A:BC' client_ip='192.168.1.100' identity='JohnDoe' aid='1687088497???

802.1x EAP De-association Message
* EAP session end
    * `<134>1 1580551705.928047208 my_AP events type=8021x_deauth radio='1' vap='2' identity='JohnDoe' aid='1687088497???'

* Note, while there is an association id (session.id) created prior to wpa/802.1x authentication, building the session event from the eap success message allows for easier integration of fields like username, client.ip, etc. in an 802.1x or WPA environment

    * Base 802.11 Association:  (802.11 session start)
        * `<134>1 1380653443.857790533 MR18 events type=association radio='1' vap='1' channel='2' rssi='23' aid='1687088497???

    * Base 802.11 Deassociation Message  (802.11 session end)
        * `1380653443.857790533 my_AP events type=disassociation radio='1' vap='2' channel='6' reason='8' instigator='2' duration='11979.728000' auth_neg_dur='1380653443.85779053324000' last_auth_ago='5.074000' is_wpa='1' full_conn='1.597000' ip_resp='1.597000' ip_src='192.168.111.251' arp_resp='1.265000' arp_src='192.168.111.251' dns_server='192.168.111.1' dns_req_rtt='1380653443.85779053335000' dns_resp='1.316000' aid='1813578850'


Example 2: ASA Admin Login
* Session start
    * `<166>Feb 03 2020 11:27:05 5508x-1_9.12(3): %ASA-6-605005: Login permitted from 192.168.1.250/59277 to management:192.168.1.10/ssh for user "JohnDoe"
* Session End
    * `<166>Feb 03 2020 11:27:05 5508x-1_9.12(3): %ASA-6-315011: SSH session from 192.168.1.250 on interface management for user JohnDoe disconnected by SSH server, reason: timeout

Example 3: ASA Web VPN
* Session Start
    * `<166>Feb 03 2020 11:27:05 5508x-1_9.12(3): %ASA-6-721016: WebVPN session for client user JohnDoe , 192.168.1.100 has been created.

* Session End:
    * `<166>Feb 03 2020 11:27:05 5508x-1_9.12(3):%ASA-6-721018: WebVPN session for client user JohnDoe , IP 192.168.1.100 has been deleted.

Example 4: (DB Connection?)
* TBD

Example 5: (Web Session?)
* TBD

Example 6: (Cloud Admin Session?)
* TBD
<!--
Stage 1: Provide a high-level description of example sources of data. This does not yet need to be a concrete example of a source document, but instead can simply describe a potential source (e.g. nginx access log). This will ultimately be fleshed out to include literal source examples in a future stage. The goal here is to identify practical sources for these fields in the real world. ~1-3 sentences or unordered list.
-->

<!--
Stage 2: Included a real world example source document. Ideally this example comes from the source(s) identified in stage 1. If not, it should replace them. The goal here is to validate the utility of these field changes in the context of a real world example. Format with the source name as a ### header and the example document in a GitHub code block with json formatting.
-->

<!--
Stage 3: Add more real world example source documents so we have at least 2 total, but ideally 3. Format as described in stage 2.
-->

## Scope of impact

Scope of impact should be minimal, with no breaking changes expected.

<!--
Stage 2: Identifies scope of impact of changes. Are breaking changes required? Should deprecation strategies be adopted? Will significant refactoring be involved? Break the impact down into:
 * Ingestion mechanisms (e.g. beats/logstash)
 * Usage mechanisms (e.g. Kibana applications, detections)
 * ECS project (e.g. docs, tooling)
The goal here is to research and understand the impact of these changes on users in the community and development teams across Elastic. 2-5 sentences each.
-->

## Concerns
- inclusion of APM / programming considerations of session to ensure compatibility, and extend if necessary
- consideration of authentication mechanisms, providers, etc (e.g. user logs into Cisco ASA and authenticates
  against ldap, radius, tacacs, specific network proteocls like eap, wpa, SAML/OAUTH, etc.)
<!--
Stage 1: Identify potential concerns, implementation challenges, or complexity. Spend some time on this. Play devil's advocate. Try to identify the sort of non-obvious challenges that tend to surface later. The goal here is to surface risks early, allow everyone the time to work through them, and ultimately document resolution for posterity's sake.
-->

<!--
Stage 2: Document new concerns or resolutions to previously listed concerns. It's not critical that all concerns have resolutions at this point, but it would be helpful if resolutions were taking shape for the most significant concerns.
-->

<!--
Stage 3: Document resolutions for all existing concerns. Any new concerns should be documented along with their resolution. The goal here is to eliminate the risk of churn and instability by resolving outstanding concerns.
-->

<!--
Stage 4: Document any new concerns and their resolution. The goal here is to eliminate risk of churn and instability by ensuring all concerns have been addressed.
-->

## Real-world implementations
Session fields would allow for the further normalization of VPN logs, application logs (e.g. ftp logs from firewalls, SQL Server sessions), administrative sessions (search for admin sessions on non encrypted ports), analyze and track user session behaviors across numerous infrastructure and application log sources.

<!--
Stage 4: Identify at least one real-world, production-ready implementation that uses these updated field definitions. An example of this might be a GA feature in an Elastic application in Kibana.
-->

## People

The following are the people that consulted on the contents of this RFC.

* @DainPerkins | Author

<!--
Who will be or has consulted on the contents of this RFC? Identify authorship and sponsorship, and optionally identify the nature of involvement of others. Link to GitHub aliases where possible. This list will likely change or grow stage after stage.

e.g.:

* @Yasmina | author
* @Monique | sponsor
* @EunJung | subject matter expert
* @JaneDoe | grammar, spelling, prose
* @Mariana
-->


## References

<!-- Insert any links appropriate to this RFC in this section. -->

### RFC Pull Requests

<!-- An RFC should link to the PRs for each of it stage advancements. -->

* Stage 0: https://github.com/elastic/ecs/pull/879

<!--
* Stage 1: https://github.com/elastic/ecs/pull/NNN
...
-->
