Email Validate API V2 Documentation
--------------------

A-1. OVERVIEW
--------------------

The Email Validation Platform provides real-time and batch validation of email
addresses. The platform is accessible via a RESTful web service as well as an automated
SFTP service, as described in Parts B and C of this specification, respectively.

A-2. VALIDITY
--------------------

The validity of an email address is defined as follows:

<table>
<tbody>
<tr><th>Value</th><th>Description</th></tr>
<tr><td>verified</td><td>Mailbox exists, is reachable, and not known to be illegitimate or disposable.</td></tr>
<tr><td>disposable</td><td>Domain is administered by disposable email provider (e.g. Mailinator).</td></tr>
<tr><td>unreachable</td><td>Domain does not exist or has no reachable mail exchangers (see discussion, below).</td></tr>
<tr><td>illegitimate</td><td>Seed, spamtrap, black hole, technical role account or inactive domain.</td></tr>
<tr><td>undeliverable</td><td>Mailbox does not exist, or mailbox is full, suspended or disabled.</td></tr>
<tr><td>unknown</td><td>We were unable to conclusively verify or invalidate this address.</td></tr>
</tbody>
</table>

A **role** account is an email address that is not associated with a specific person, but rather
with an office, position, group or task. Role accounts can be broadly classified as either
*technical* or *nontechnical*.

- A *technical* role account has a local part such as *postmaster*, *abuse* or *spam*.
Because such accounts should never be mailed, these addresses are classified as **illegitimate**.

- A *nontechnical* role account has a local part such as *sales*, *support*, *info*, *legal*, or
*inquiries*. Whether you choose to accept such addresses depends on your use case,
and may require case-by-case, manual evaluation. For these addresses, the value
of the **role** field will be **true**.

A **full** mailbox is **undeliverable**, and typically indicative of an inactive account.
However, if you would like to accept these, you can identify them by the existence of the 
**full** field, containing a value of **true**.

An email address is **unreachable** if either no mail exchangers are advertised for its
domain or they all persistently reject or do not reply to inbound connections. While almost
always indicative of a domain which is not used to receive email, transient configuration or
DNS errors on the receiving end may also be at fault.


A-3. CORRECTIONS
--------------------

Whenever possible, the Email Validation platform will provide suggested corrections for addresses 
that have been deemed invalid. Corrections are handled slightly differently by the RESTful web 
service and SFTP batch processing components, and these specifics are described in sections B-9 
and C-6, respectively.

All corrections returned as part of the services are provided as suggestions only. This is not a 
guarantee that the email addresses are correct nor that they can be used in any specific manner or 
are email addresses you are authorized to send to. Please check with your legal department to 
confirm that the manner in which you plan to use the emails prior to sending email to such 
recipients.


B-1. WEB SERVICE OVERVIEW
--------------------

The Email Validation Platform is accessible via a RESTful web service for real-time
validation. API resources are identified by URLs and interacted with using the standard
HTTP methods: GET, PUT, POST and DELETE.

An illustrative example of a real-time query and its result is given in *Section* B-10.

B-2. VERSIONING
--------------------

The client specifies the desired API version in the URL of each request (see *Section* B-5)
so that new versions do not break existing deployments. Whenever the introduction of
compatibility-breaking changes is necessary, a new version number will be allocated, but
this should be relatively infrequent. The most recent two API versions will be supported
at any given time, but only the newest version will be actively maintained.

B-3. SECURITY
--------------------

All API traffic is encrypted via TLS (SSL) and the servers are authenticated via 2048-bit
X.509 digital certificates. Client-side (JavaScript) queries are authorized via the HTTP
Referer (sic) header (*N.B.* in order to prevent theft of service by third parties via header
spoofing, only a limited number of Referer (sic)-authorized queries are accepted per IP
address). Server-side queries are authenticated via an API-key URL parameter.

B-4. AVAILABILITY
--------------------

In order to promote service availability even in the event of a data center-wide disruption,
two API servers are provided, situated in geographically distinct data centers. Requests
may be directed to either server. No appreciable difference in performance or behavior
should be expected. Clients are expected to alternate API servers in the event of
communication failure or persistent errors.

B-5. URL STRUCTURE
--------------------

All resources are exposed according to the following URL structure:

`https://{server}/{version}/path/to/resource`

Required parameters:

<table>
<tbody>
<tr><th>Name</th><th>Description</th><th>Value(s)</th></tr>
<tr><td>server</td><td>The server name.</td><td>api2.qasemail.qas.com</td></tr>
<tr><td>version</td><td>The API version.</td><td>v2</td></tr>
</tbody>
</table>

B-6. STATUS CODES
--------------------

Clients must examine the HTTP status code of server replies before attempting to
interpret their content. The following status codes are specifically defined, but clients
should be prepared to interpret other status codes, such as might be generated by
intermediate proxies or firewalls, as per RFC 2616:

<table>
<tbody>
<tr><th>Status code</th><th>Interpretation</th></tr>
<tr><td>200 OK</td><td>The request was well-formed and the reply contains the result.</td></tr>
<tr><td>202 Accepted</td><td>The request was well-formed but the result is not yet available.</td></tr>
<tr><td>400 Bad Request</td><td>The request was malformed and should not be retried without modification. The reply content will contain a description of the error.</td></tr>
<tr><td>401 Unauthorized</td><td>No authorization credentials were supplied, they were invalid, or they were encoded incorrectly. Check your WWW-Authenticate header.</td></tr>
<tr><td>500 Internal Server Error</td><td>An unexpected error occurred. The request may be retried (perhaps with an alternate server), but persistent failures should be reported.</td></tr>
<tr><td>503 Service Unavailable</td><td>The server is temporarily overloaded and not accepting new requests. Requests should temporarily be directed to an alternate server.</td></tr>
</tbody>
</table>

B-7. REPRESENTATIONS
--------------------

The default resource representation format is JSON; however, XML representations can
be requested by specifying application-xml in the Accept header. Clients unable
to modify headers may pass this value via an accept-header URI parameter, instead.

When validating server replies, clients should ensure that all expected fields are present.
However, in order to accommodate intra-version additions to the API, any unexpected
fields must be ignored. Clients must not rely on any undocumented fields.

B-8. REAL-TIME VALIDITY RESOURCE
--------------------

GET is the only method allowed for this resource. Its relative path has the following
structure:

`/validity/{address}?timeout={timeout}`

There are two required parameters:

<table>
<tbody>
<tr><th>Name</th><th>Description</th><th>Value</th></tr>
<tr><td>address</td><td>The email address to validate.</td><td>A percent-encoded email address.</td></tr>
<tr><td>timeout</td><td>The timeout, in seconds.</td><td>Integer (truncated to the range [3, 15]).</td></tr>
</tbody>
</table>

The representation (named validity, if XML) will contain the following fields:

<table>
<tbody>
<tr><th>Name</th><th>Description</th><th>Value(s)</th></tr>
<tr><td>address</td><td>The email address.</td><td>A string (nominally an email address).</td></tr>
<tr><td>result</td><td>The validation result.</td><td>See Section A-2.</td></tr>
</tbody>
</table>

The representation may also contain one or more of the following fields:

<table>
<tbody>
<tr><th>Name</th><th>Description</th><th>Value(s)</th></tr>
<tr><td>role</td><td>Is this a nontechnical role account?</td><td>'true' or not present.</td></tr>
<tr><td>full</td><td>Is the mailbox full?</td><td>'true' or not present.</td></tr>
<tr><td>timeout</td><td>Was the query's timeout exhausted?</td><td>'true' or not present.</td></tr>
<tr><td>retry</td><td>You may retry after this many seconds.</td><td>An integer or not present.</td></tr>
<tr><td>corrections</td><td>List of suggested corrections.</td><td>Array consisting of email addresses.</td></tr>
</tbody>
</table>

Clients will receive a result of unknown for any queries not satisfied within the
timeout period, starting from when the request is received, and ending when it is replied
to, server-side.

Results of unknown or unreachable may occasionally be due to transient issues, such
as DNS errors, mail exchanger misconfiguration, server rate-limiting (for real-time
queries) or even internal errors. When there is any possibility of this being the case, the
result field will be accompanied by a retry field, specifying how many seconds must
be waited before retrying the query, if desired.

B-9. REAL-TIME CORRECTIONS
--------------------

Zero or more corrections may be returned for a given email address. Corrections will be returned by the real-time endpoint as an array of email addresses, sorted by descending order of likelihood.

Corrections returned by real-time validation are not guaranteed to be verified. The Email Validation platform makes a best-effort to validate these suggested corrections without affecting response time. The system attempts to validate all suggested corrections simultaneously, removing any invalid addresses as they are identified. When the original address has been conclusively validated, the array of remaining corrections is returned.

B-10. REAL-TIME VALIDITY RESOURCE EXAMPLES
--------------------

To validate "fake-address@example.com", the client might GET the resource at:

`https://api2.qasemail.qas.com/v2/validity/fake-address@example.com?timeout=5&key={API key}`

and would receive the following representation:

`{ "address": "fake-address@example.com",
 "result": "undeliverable" }

#### Example With Corrections

To validate "fake address@gmal.com," the client might GET the resource at:

`https://api2.qasemail.qas.com/v2/validity/fake%20address@gmal.com?timeout=5&key={API key}`

the response would be in the form of:

'{ “address”: “fake address@gmal.com”,
“result”: “undeliverable”,  
"corrections" : ["fakeaddress@gmail.com", "fakeaddress@gmal.com"] }

*NOTE: the corrections shown above are purely for documentation purpose and may not reflect the 
behavior of the corrections module.*

 
C-1. FTP SERVICE OVERVIEW
--------------------

The Email Validation Platform is accessible via an FTP service for batch processing.
Special folders (see *Section* C-4) and error-checking (see *Section* C-5) facilitate
robust, fully automated integrations.

C-2. SECURITY
--------------------

All files are transmitted via TLS (SSL) and the servers authenticated via 2048-bit X.509 
digital certificates.

No client's files nor folders are visible to nor accessible by any other client.

C-3. CONNECTING
--------------------

To connect to the SFTP server, the following configuration must be used, along with your 
account username and password:

<table>
<tbody>
<tr><th>Name</th><th>Description</th><th>Value(s)</th></tr>
<tr><td>Host</td><td>URL for the SFTP server.</td><td>sftp2.qasemail.qas.com</td></tr>
<tr><td>Port</td><td>Port to use for connections.</td><td>22</td></tr>
</tbody>
</table>

C-4. FOLDERS
--------------------

The client has access to the following folders:

<table>
<tbody>
<tr><th>Path</th><th>Description</th></tr>
<tr><td>incoming</td><td>Place files here to submit them for validation.</td></tr>
<tr><td>outgoing</td><td>Pick up completed files here.</td></tr>
</tbody>
</table>

The client has the following filesystem permissions and restrictions:

- Can only upload files into the incoming folder.
- Cannot move/rename any files.
- May delete files from any folder.
- Cannot create, rename or remove any folders.

C-5. INPUT
--------------------

Uploaded files must be RFC 4180-compliant CSVs (*cf.* http://tools.ietf.org/html/rfc4180 ):

- Commas are used to separate fields.
- DOS/Windows-style newline (CR+LF) are used to terminate records.
- The last record may or may not be terminated.
- Double-quotation marks are used to quote fields.
- Double-quotation marks are used to escape double-quotation marks.
- Fields containing commas, newlines or double-quotation marks must be quoted.
- Otherwise, quoting is optional.

In addition to the above, a header row is required. The email-address column must be
named *email* (case-insensitive). Additional columns may be included, and will be
preserved in the output, but none of them may be named result or correction.
Additional column names may be reserved, in the future.

ASCII-mode FTP converts newline sequences during transfer, so use binary mode!

C-6. OUTPUT
--------------------

Output files will adhere to the same format as used for input, except containing additional
fields named result and correction (with values as described in *Section* B-8). 

Unlike the real-time API, if more than one correction is suggested by the service, only the most 
likely correction will be returned. This correction has been validated, and is guaranteed to be a
"verified" or "unknown" result. 

To simplify parsing (*e.g.* for direct, MS-SQL bulk import ), *every* field will be quoted.

C-7. AUTOMATION
--------------------

It is helpful to think of the FTP service as a *state machine*: Whichever folder a file is in
reflects its current processing *state*.

Importantly, because a file is only in a single state at any given time, it is not necessary to
poll every folder. Rather, having uploaded a file, one need *only* poll the incoming
folder. When the file is no longer there, it will appear in the outgoing directory for download.

Improperly formatted files will are rejected from the system during the upload process. You will receive an email notification with details about what was wrong with the file. If malformed, manual intervention is required&#8212;something is wrong with how the file is formatted, and re-uploading it will not help. Review *Section* C-6 and ensure you have included a proper header row, are quoting any special characters (and properly escaping any quotes), are using the correct line terminators, and are using binary transfer mode.

When a file has completed processing, you will receive an email notification. The results may be retrieved from the outgoing folder, and will remain there for at least 30 days. After 30 days they may be deleted automatically by the FTP service due to our data retention policy. It is the client's responsibility to collect results before this time&#8212;and likewise to delete them, if so desired.

Before acting on the results, the client should always ensure they conform to the expected
format, contain the expected records, and the results are within reasonable bounds.
