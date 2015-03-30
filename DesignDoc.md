# Table of Contents #


# Capirca Design Doc #

Status: Final<br>
Author: Tony Watson<br>
Created: Nov 2, 2007<br>
Last Updated: May 5, 2010<br>

<h2>Objective</h2>

Define a common meta-language to describe security policies, and a standardized interconnect between meta-language and actual policy rules.  The meta-language should be flexible enough to support most common network access control (NAC) devices, but simple enough to be clear and easy to understand.  The interconnect should provide a common understanding of how and where the meta-language and actual policy rules are stored.<br>
<br>
<h2>Goals</h2>

<ul><li>Provide a standard meta-language to describe NAC policies<br>
</li><li>Avoid the proliferation of differing ACL meta-language formats<br>
</li><li>Provide a common framework for maintaining both meta-language policies and the actual applied NAC policy<br>
</li><li>Provide a foundation for expanding automation of NAC processes<br>
</li><li>Eliminate confusion and guesswork when implementing a new output format generator</li></ul>


<h2>Background</h2>

Currently, the security group utilizes a variety of tools to automate the generation of ACL, F10, JCL, and Iptables policies.  Historically, many automation tools have been built using Ruby around the naming.rb library.  As these tools have been developed they have usually had unique limitations or requirements that resulted in slightly differing input and output formats.  The problem is not serious today, but must be resolved soon in order to avoid serious headaches in the future.<br>
<br>
<h2>Problems</h2>

A standardized model is needed to bring existing tools into a happy co-existence, as well as to prevent continued deviations in future tools.  The following is a list of some of the existing concerns:<br>
<br>
<ul><li>JCL meta-policy is embedded within comments inside the actual JCL policy files.  The resulting output simply replaces in-line terms and replaces the original input file with the generated output file.  Meta-policies are maintained in comments immediately after the policies own 'term' statement and any non-replaced lines in the policy are appended verbatim to the output.<br>
</li><li>Speedway uses separate meta-policy and generated iptables policy files.  Meta-policies are parsed and the output sent to the policy module files in another directory.  Speedway meta-policy defines new policies using the 'policy' keyword and all other content in the meta is appended verbatim to the output.<br>
</li><li>F10 meta-policy uses separate meta-policy and generated ACL policy files.  Meta-policies are parsed and the output sent to the policy module files in the same directory.  F10 generator meta-policy defines new policies using the 'term' keyword and all non-term content is ignored.</li></ul>

<table><thead><th> Generator Type </th><th> Meta-Policy Definition Location </th><th> New Policy Keyword </th><th> File Naming Standards </th><th> Comments and Non-Meta-Policy Lines </th></thead><tbody>
<tr><td> Juniper </td><td> inline </td><td> uses existing policy 'term' statement </td><td> inline with .jcl files </td><td> non-replaced term lines are appended verbatim to output </td></tr>
<tr><td> Speedway </td><td> separate files, different directories </td><td> policy xyz { </td><td> meta-policy filename mirrors generated policy in different directories </td><td> non meta-policy lines are appended verbatim to output </td></tr>
<tr><td> Cisco </td><td> separate files, same directory </td><td> term xyz { </td><td> meta-policy and generated policy have .pol and .acl extension in same directory </td><td> non meta-policy lines are ignored </td></tr>
<tr><td> others ... </td><td>  </td><td>  </td><td>  </td><td>  </td></tr></tbody></table>

<h2>Meta-Policy Integration</h2>


<b>Files:</b><br>
The policy file will consist of terms, comments, and other directives.  Terms are the specific rules relating a variety of properties such as source/destination addresses, protocols, ports, actions, etc.  Directives may be used to specific that a particular policy should only be generated for a specific platform target, such as cisco.  The policy file has the following properties:<br>
<ul><li>Comments are ignored.  They are not accessible outside the policy class.<br>
</li><li>Directives are acted on by the compiler.  They may be accessible through the policy class.<br>
</li><li>Terms are interpreted and used to populate the policy object.</li></ul>

<b>File Names and Locations:</b><br>
The policy files shall be named appropriately to describe their functionality and purpose.  Policy files will use a .pol file extension.  The network ACL perforce repository will maintain separate 'pol' sub-directory beneath each 'acl' directory, to contain the policy description file and the generated filters respectively.  The following diagram illustrates the suggested directory structure:<br>
<br>
<pre><code> <br>
                      ./network/acl<br>
                            |<br>
 -+------------+------------+------------+-<br>
  |            |            |            |<br>
 Def         Corp          Prod        Sysops<br>
  |            |            |            |<br>
 -+------------+------------+------------+-<br>
               |            |            |               <br>
             Policy        Policy      Policy          <br>
<br>
</code></pre>

Generated output will be stored in files with identical filenames, but lacking the .pol extension.<br>
<br>
<br>
<h2>Policy Description Language Definition</h2>

The  NAC team needs a standardized meta-policy that can support a wide variety of platforms such as Cisco, Juniper, F10, Netscreen, and Iptables.  The language needs to be flexible enough to support diverse platforms, but rigid enough in its definition to ensure consistency between policy definitions.<br>
<br>
Each policy description file consists of one or more sections.  Each section must begin with a 'header' block, followed by one or more 'term' blocks.<br>
<br>
<h3>Header Description Format</h3>

The header section is used to specify options that apply to all terms blocks within the policy, such as the target output platform and any arguments needed by output platform generator.<br>
<br>
<pre><code> comment:: [doublequoted text]3<br>
 target:: [platform] [arguments]<br>
</code></pre>

The arguments for each platform are passed directly to the output generator as a list, and vary depending on the needs of the generator.  Below is a list of currently supported generators and their argument lists.  Arguments in <a href='.md'>.md</a>'s are required, arguments in {}'s are optional.<br>
<br>
<pre><code>  target:: cisco [named-access-list] {extended|standard|object-group|inet6|mixed}<br>
  target:: juniper [filter-name] {inet | inet6 | bridge}[1]<br>
  target:: iptables [INPUT | OUTPUT | FORWARD] {ACCEPT | DROP}[2]<br>
</code></pre>

<code> [1] </code> The juniper generator defaults to inet (ipv4) output, but ipv6 and bridge filters can also be specified in the optional filter_type argument.<br>
<code> [2] </code> The iptables generator target must specify a filter which the terms will apply to.  The optional 'default action' of ACCEPT or DROP may be used to include output to set the default action of the named filter.<br>
<br>
<h4>Example</h4>
<pre><code>  header {<br>
    comment:: "This is an example header "<br>
    comment:: "used in policy definition files..."<br>
    target:: juniper inbound-edge-filter inet6<br>
    target:: iptables INPUT DROP<br>
  }<br>
</code></pre>

<h3>Term Definition Format</h3>
Terms follow the<br>
<br>
<blockquote>Tokens / keywords that must be supported.<br>
</blockquote><ul><li>source-address::  <a href='token.md'>token</a>
</li><li>source-exclude::  <a href='token.md'>token</a>
</li><li>destination-address::  <a href='token.md'>token</a>
</li><li>destination-exclude::  <a href='token.md'>token</a>
</li><li>source-port::  <a href='token.md'>token</a>
</li><li>destination-port::  <a href='token.md'>token</a>
</li><li>protocol::  [tcp,udp,icmp, or protocol #]<br>
</li><li>action::  [accept/reject/deny/next]<br>
</li><li>option::  [established, sample, rst, initial, other arbitrary user supplied]<br>
</li><li>verbatim:: <a href='target.md'>platform</a> <a href='doublequoted.md'>text field</a></li></ul>

Tokens / keywords that may be supported.<br>
<ul><li>packet-length:: [text,None (default None)]<br>
</li><li>fragment-offset:: [text,None (default None)]<br>
</li><li>counter::  [text,None (default None)]<br>
</li><li>policer::  [text,None (default None)]<br>
</li><li>logging::  [text,None(default)]<br>
</li><li>direction:: [inbound, outbound, both(default)]<br>
</li><li>qos::     (text,None (default None)    for juniper = forwarding-class)<br>
</li><li>target:: [juniper, cisco, iptables] <a href='filter.md'>type; inet, inet6, bridge (juniper  specific)</a> [options; default filter action in iptables]<br>
</li><li>comment:: "doublequoted text field"<br>
</li><li>source-prefix:: "text" (prefix lists are used in juniper and are comparable to address directives except that they're defined on the router itself)<br>
</li><li>destination-prefix:: "text"</li></ul>

<ul><li>Policy files should render equivalent output for any given renderer/target.<br>
</li><li>Generators may not support all keywords, they can ignore keywords as desired but should produce warnings.<br>
</li><li>Generators must produce equivalent access lists from the same policy file.<br>
</li><li>Documentation comments consist of any hash mark (#) through EOL and should be passed to generators in the order they appear in the meta policy.<br>
</li><li>Generators should ignore comments.<br>
</li><li>Per term comments in meta-policy can be included in sections such as header and terms, using the following notation:  comment:: "<a href='text.md'>text</a>". All text between double quotes, including newlines, becomes the comment<br>
</li><li>Terms in meta-policy will be indicated by opening and closing identifiers:  term x { .... }<br>
</li><li>A header section shall begin each meta-policy.  The header section shall be denoted by the following notation:  header { ... }<br>
</li><li>A header must contain at least one target:: section, which specifies the platform or platforms for which the following terms will be rendered<br>
</li><li>A header section may span multiple lines.<br>
</li><li>A header may contain a comment section, denoted as comment:: "<a href='text.md'>text</a>"<br>
</li><li>The option 'established' shall imply adding high-ports to terms with TCP or UDP only protocols, tcp-flag checking on TCP only terms, and may imply stateful checking for generators that support it.<br>
</li><li>The option 'tcp-established' shall imply tcp-flag checking for terms where only the TCP protocol is specified.  It may imply stateful checking for generators that support it.<br>
</li><li>other?<br>
<h3>Policy Object</h3></li></ul>

A policy object is collection of sections, such as header and terms, as well as their associated properties.  Each section includes a variety of properties such as source/destination addresses, protocols, ports, actions, etc.<br>
<br>
The policy.py module generates policy objects from policy files.<br>
<br>
<h4>ParsePolicy</h4>

A policy object can be created by passing a string containing a policy to the ParsePolicy() class.<br>
<br>
<pre><code>  policy = policy.ParsePolicy(policy_text)<br>
</code></pre>

<h4>Headers</h4>

<pre><code>  for header, terms in policy.filters:<br>
      header.target<br>
      header.target.filter_name<br>
</code></pre>

<h4>Terms</h4>

<pre><code>  for header, terms in policy.filters:<br>
    terms[x].action[]<br>
<br>
    # addresses - lists of google3.ops.security.lib.nacaddr objects<br>
    terms[x].address[]<br>
    terms[x].destination_address[]<br>
    terms[x].destination_address_exclude[]<br>
    terms[x].source_address[]<br>
    terms[x].source_address_exclude[]<br>
<br>
    # ports - list of tuples.<br>
    terms[x].port[]<br>
    terms[x].destination_port[]<br>
    terms[x].source_port[]<br>
<br>
    # list of strings<br>
    terms[x].comment[]<br>
    terms[x].protocol[]<br>
    terms[x].option[]<br>
    terms[x].verbatim[x].value<br>
<br>
    # string<br>
    terms[x].counter<br>
    terms[x].name<br>
<br>
</code></pre>
<h4>Example</h4>

a contrived example follows:<br>
<pre><code>  header { <br>
    comment:: "this is an example filter"<br>
    target:: junniper example-filter<br>
    target:: cisco example-filter inet<br>
  }<br>
<br>
<br>
  term term-1 {<br>
    source-address:: BIG_NETWORK<br>
    destination-address:: BIG_NETWORK<br>
    protocol:: tcp<br>
    action:: accept<br>
  }<br>
</code></pre>

this would output a juniper filter of:<br>
<pre><code>family inet {<br>
    replace: <br>
    filter example-filter {<br>
        interface-specific;<br>
        term term-1 {<br>
            from { <br>
                source-address {<br>
                    10.0.0.0/8;<br>
                }<br>
                destination-address {<br>
                    10.0.0.0/8;<br>
                }<br>
                protocol tcp;<br>
            }<br>
            then {<br>
                accept;<br>
            }<br>
        }<br>
    }<br>
}<br>
</code></pre>

and a cisco filter of:<br>
<pre><code>no ip access-list extended example-filter<br>
ip access-list extended example-filter<br>
<br>
permit tcp 10.0.0.0 0.255.255.255 10.0.0.0 0.255.255.255<br>
</code></pre>

<h4>IPv6</h4>


IPv6 support has been added to the policy language.  Currently only Cisco, Juniper and Iptables can render ipv6 filters.  The syntax for an ipv6 filter is exactly the same as ipv4 filters except for the inet6 keyword on the target line.  Making an ipv6 filter is as easy as<br>
<br>
<pre><code>  header {<br>
    target:: juniper some-v6-filter inet6<br>
  }<br>
</code></pre>

Be sure that the addresses you reference in your subsequent terms have ipv6 definitions.  ie, if you have<br>
<br>
<pre><code>  term my-v6-term {<br>
    source-address:: PRODUCTION_NETWORK<br>
    destination-address:: CORPORATE_NETWORK<br>
    protocol:: tcp<br>
    action:: accept<br>
  }<br>
</code></pre>

When PRODUCTION_NETWORK or CORPORATE_NETWORK tokens are only defined with ipv4 addresses, this will error out.  Tokens can include both IPv4 and IPv6 addresses, and rendering IPv6 output will include only IPv6 addresses associated with a given token.<br>
<br>
<br>
<h3>Definitions</h3>

The following are words that we have defined for the purposes of NAC discourse and this project. Some of these words may be defined somewhat differently than you are used to.<br>
<br>
Generator:  A program that utilizes the data contained in a Policy to create an output rulebase suitable for applying to a specific target platform.  Generators will be specific for each target platform, such as juniper, cisco, f10, iptables, etc.<br>
<br>
Global Directive:  Keywords contained outside of a term or comment within a policy, that define a default value for a particular term property.  Global directives can be overwritten within an individual term by specific redefinition within the term.  Global directives are limited to only those keywords allowed within a term definition.<br>
<br>
NAC: Network Access Control. Concerning issues related to security at layer 3 and 4 in the OSI model.<br>
<br>
Flow: A network flow, given as a tuple of the following form:  (src(s), dst(s), src-port(s), dst-port(s), protocol)<br>
<br>
Service: A set of tuples of the form ((server/network), port, protocol, <a href='application.md'>application</a> ) that share a common logical function.<br>
<br>
Term: A flow to/from a service, the action applied to this flow, and where this action is enforced (e.g., what PEP(s)).  A term is expressed in the form of a tuple:<br>
(src(s), dst(s), src-port(s), dst-port(s), protocol, action = {permit/drop/deny, etc.}, modifier(s) = {QOS, negation, counters, etc.}, PEP(s) )<br>
<br>
Policy: A policy is the set of all terms which apply to a particular service.<br>
<br>
Rule: A rule is the device-specific implementation of a term.<br>
<br>
Rulebase: A rulebase is the set of all rules on a device.<br>
<br>
Logical Rule: The set of all device-specific implementations of a term.<br>
<br>
Logical Rulebase: The set of all device-specific implementations of terms pertaining to a particular policy.<br>
<br>
Narrative: The narrative is the English language description of a given service's policy, along with the justification for these policies and meta-information about the service. (Who is authorized to make changes, what the procedure is for making changes, etc.). One requirement is that the English language description is sufficiently unique that we have a mapping between section of narrative -> terms of policy -> rules of rulebase.<br>
<br>
PEP: Policy enforcement point.  A PEP is any location on the network where the terms of a policy can be enforced as rules.<br>
<br>
LPEP: Logical PEP.  The set of all devices and/or interfaces that enforce a logical security boundary.