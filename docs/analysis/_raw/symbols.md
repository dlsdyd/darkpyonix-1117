# 심볼 추출 (자동)
형식: `줄 종류 이름: 첫 doc`. H는 파일 헤더 주석.

## darkpyonix/hub/server/src/lib.rs (206)
H: darkpyonix.dev, the DarkPyonix hub (SPEC §10, docs/api/hub.openapi.yaml). Partly superseded (INTENT D15, 2026-10-03).** The hub API now runs on Cloudflare Workers (`darkpyonix/hub/worker/`): GitHub accounts and device li…
28 pub struct TlsCert: A certificate chain and its private key for the hub's own TLS.
34 pub struct HubConfig: How to run a hub.
55 pub enum HubError: Errors starting a hub.
69 pub struct RunningHub: A running hub. Dropping it stops every listener.
78  impl RunningHub: 
100  impl Drop for RunningHub: 
108  fn server_tls_config: 
118 pub fn start: Starts a hub and returns once every listener is bound.

## darkpyonix/hub/server/src/main.rs (84)
H: `darkpyonix-hub`: the darkpyonix.dev server binary.
18  struct Args: darkpyonix.dev: device registry, address directory, iroh relay, share links, HTTPS names.
45  fn load_tls: 
52  fn main: 

## darkpyonix/hub/server/src/routes.rs (51)
H: HTTP probes and relay administration.
9 pub fn router: 
15 pub fn side_router: 
25  fn health: 
26  fn ping: 
27  fn generate_204: 
34  fn authorize: 
44  struct Disconnect: 
45  fn disconnect: 

## darkpyonix/hub/server/src/serve.rs (92)
H: One listener for everything: TLS (optional), then HTTP/1.1. `GET /relay` goes to the embedded iroh relay service, every other request to the axum router.
26 pub(crate) struct Frontend: 
31  impl Frontend: 
53 pub(crate) fn run: Accepts connections until the task is aborted.

## darkpyonix/hub/server/src/state.rs (91)
H: Relay admission and presence (SPEC FR-H3).
9 pub(crate) struct State: 
20  struct Admission: 
21  impl State: 
80 pub(crate) struct HubAccess: 
81  impl fmt::Debug for HubAccess: 
82  impl AccessControl for HubAccess: 

## darkpyonix/hub/worker/src/app.ts (202)
H: Routing for darkpyonix.dev (docs/api/hub.openapi.yaml). Every operation of that file that is served by this Worker is listed here, and nothing else (CLAUDE.md: the OpenAPI is the SPEC).
30 export const VERSION: 
43  function route: 
50  function health: 
55  function config: `GET /config` (FR-H8): where the relays and the pkarr directory are. No API version (D16). */
75  function flathubVerifiedApps: `GET /.well-known/org.flathub.VerifiedApps.txt` (FR-H7): Flathub's website verification file, the to…
85  function linkLanding: `GET /link?code=...`: the approval page; signs the browser in with GitHub first. */
104 export const ROUTES: 
150  function dispatch: 
172 export function handle: 
193 export function cleanup: Hourly: drop expired single-use rows. */

## darkpyonix/hub/worker/src/devices.ts (622)
H: Accounts' devices (SPEC FR-H1): device links, the device list, removal. A device joins an account through a device link, the shape of OAuth device authorization (RFC 8628) with key possession added: 1. the device asks: P…
40 export const LINK_TTL_SECS: How long a person has to approve and the device to claim. */
42 export const LINK_POLL_INTERVAL_SECS: How often a device should poll for its token. */
44 export interface DeviceRow: 
57 export interface DeviceApp: What a device says it runs (FR-H10); self-reported, a hint only. */
63 export function deviceJson: 
80 export function parseDeviceApp: A DeviceApp from untrusted JSON, or null if it is not one exactly. */
93 export function linkMessage: 
98 export const ROLES: Device roles (SPEC FR-H1). `client` only connects to other devices (provisional). */
99 export type Role: 
101  function validRole: 
105  function validDeviceName: 
109  function limited: 
115  function clientIp: 
122 export const MAX_WAIT_SECS: Longest `wait` of a long-poll on `GET /devices`. */
124 export const WAIT_POLL_MS: How often a long-poll re-reads the account's version. */
127 export function bumpDevicesVersion: Marks a change visible in the account's device list (not `last_seen` alone). */
131  function devicesVersion: 
138  function etagFor: 
143  function ifNoneMatches: Weak comparison (RFC 9110 §13.1.2): `W/` is ignored; `*` matches. */
151 export function me: 
167 export function createLink: 
228  function currentMainServer: The account's one active main server (FR-H1), if any. */
236  function pendingLinkByCode: 
247 export function getLinkCode: `GET /link-codes/{user_code}`: what the approver is about to let in. */
265 export function decideLinkCode: `POST /link-codes/{user_code}` `{"approve": bool, "replace"?: "<endpoint_id>"}` */
316 export function getLink: `GET /device-links/{link_id}`: the link's status, for a device that restarted while waiting. */
338 export function claimLink: `POST /device-links/{link_id}/token` `{"signature": "<128 hex>"}` */
435 export function rotateResolveToken: `POST /me/resolve-token`: a new read-only resolve token; the old one stops working (NFR-H2). */
450 export function listDevices: `GET /devices[?wait=<secs>]` with `If-None-Match`: 304 when unchanged; with `wait`, held until the a…
480 export function accountDevice: 
490 export function getDevice: 
496  function requireSelfOrAccountAdmin: The device itself, or account rights over it; else 403. Returns the caller's account. */
504 export function updateDevice: `PATCH /devices/{endpoint_id}` `{"name"?: string, "app"?: DeviceApp | null}`. `name`: the device its…
539 export function readmitDevice: `POST /devices/{endpoint_id}/readmit`: the owner lets a removed key link again (FR-H11). */
556 export const MAX_REMOVED_LISTED: Most removed devices `GET /removed-devices` lists (FR-H11). */
559 export function listRemovedDevices: `GET /removed-devices`: the account's removed devices, newest first, for re-admission (FR-H11). */
586  function disconnectFromRelay: Tells the relay host to drop a removed device's connections (best effort; SPEC FR-H3). */
597 export function removeDevice: 

## darkpyonix/hub/worker/src/directory.ts (87)
H: Address directory (SPEC FR-H2): iroh's pkarr relay protocol plus a JSON view.
9  function limited: 
16 export function pkarrPut: `PUT /pkarr/{z32}`: body is the relay payload `sig(64) || ts_us(8) || dns`. */
45  function storedPayload: 
55 export function pkarrGet: `GET /pkarr/{z32}`: same-account callers only; iroh's resolver sends a resolve token as `?token=`. *…
66 export function storedAddresses: The decoded addresses of a stored record (the payload was verified when it was stored). */
76 export function deviceAddresses: `GET /devices/{endpoint_id}/addresses` */

## darkpyonix/hub/worker/src/dns.ts (106)
H: Where ACME DNS-01 TXT records go (SPEC FR-H5): the Cloudflare DNS API for darkpyonix.dev.
3 export class DnsError: 
5 export interface DnsProvider: 
32 export class CloudflareDns: The Cloudflare DNS API. The token needs only `Zone → DNS → Edit` on the one zone. Replacement is one…
93 export class MemoryDns: TXT records in memory, for tests. */

## darkpyonix/hub/worker/src/ed25519.ts (18)
H: ed25519 signature checks with Web Crypto (workerd implements the "Ed25519" algorithm).
3 export function verifyEd25519: 

## darkpyonix/hub/worker/src/env.ts (62)
H: Bindings, configuration and injectable dependencies of the hub Worker (SPEC §10).
7 export interface Env: 
51 export interface Deps: What tests replace: the network, the clock and the DNS provider. */

## darkpyonix/hub/worker/src/github.ts (210)
H: GitHub sign-in (SPEC FR-H6): OAuth 2.0 authorization code with PKCE (S256) and state. The hub asks for no scope: an unscoped GitHub token reads only the public profile, which is all we need (the numeric user id). The acc…
27 export const SESSION_TTL_SECS: 
29  function configured: 
36  function callbackUrl: 
41 export function login: `GET /auth/login?return_to=/path`: start the GitHub authorization. */
70  function exchangeCode: 
95  function fetchUser: 
112  function revokeToken: Revokes the access token (best effort): the hub keeps no GitHub credential. */
128  function allowed: 
134  function accountFor: Finds or creates the account of a GitHub user. */
160 export function callback: `GET /auth/callback?code=...&state=...`: finish sign-in and set the session cookie. */
200 export function logout: `POST /auth/logout` */

## darkpyonix/hub/worker/src/http.ts (208)
H: Responses, errors, cookies, and who is calling. Machine-readable error codes (components/schemas/Error in the OpenAPI file). An error answered as `{"error": "...", "code"?: "..."}` with a status the OpenAPI file document…
7 export type ErrorCode: Machine-readable error codes (components/schemas/Error in the OpenAPI file). */
10 export class ApiError: An error answered as `{"error": "...", "code"?: "..."}` with a status the OpenAPI file documents. */
47 export function json: 
54 export function noContent: 
58 export function html: 
70 export function redirect: 
76 export function readJson: 
88 export const SESSION_COOKIE: 
89 export const STATE_COOKIE: 
91 export function getCookie: 
102 export function setCookie: `__Host-` cookies: Secure, Path=/, no Domain. */
106 export function clearCookie: 
112 export type Principal: 
116  function bearer: 
128 export function principal: The caller, from a device token (`Authorization: Bearer`, never the query; NFR-H2) or the browser se…
170 export function requireAccountAdmin: Account-level rights (approve device links, remove devices, release names): a signed-in browser sess…
176 export const RESOLVE_TOKEN_PREFIX: Resolve tokens (NFR-H2) start with this; they are credentials only for `GET /pkarr/{key}`. */
182 export function resolvePrincipal: The caller of `GET /pkarr/{key}`: a resolve token (query or header), or else the usual header/cookie…
200 export function requireDevice: 
206 export function corsOrigin: Origins allowed for credentialed browser access (FR-H13). */

## darkpyonix/hub/worker/src/index.ts (24)
H: Worker entry for darkpyonix.dev (SPEC §10).
7  function deps: 

## darkpyonix/hub/worker/src/names.ts (109)
H: HTTPS names (SPEC FR-H5): `<name>.<zone>` reserved by a main server, and the ACME DNS-01 TXT values it asks us to publish through the Cloudflare DNS API. The main server runs its own ACME client and keeps its private key…
16  function nameJson: 
20 export function acmeFqdn: 
25 export function clearNameRecords: Clears a name's challenge records (best effort; used on release and device removal). */
31 export function listNames: 
40 export function reserveName: `PUT /names/{name}` with the main server's device token. */
56  function ownedName: The name if the caller may manage it: its device, or (when `accountMay`) its account. */
67 export function releaseName: 
75  function provider: 
81  function dnsFailure: 
87 export function setAcmeChallenge: `PUT /names/{name}/acme-challenge` `{"values": ["<43 base64url>", ...]}` */
104 export function clearAcmeChallenge: 

## darkpyonix/hub/worker/src/notebooks.ts (212)
H: FR-H19 limits. Every value is provisional ("a first value, to be changed after real use"): change them here and nowhere else. Sizes are bytes; windows are seconds.
9 export const LIMITS: FR-H19 limits. Every value is provisional ("a first value, to be changed after real use"): change th…
29  function find: 
34  function caller: 
36  function limit: 
39  function readable: 
49  function owned: 
56  function fields: 
63  function checkShare: FR-H17: the share must be published by a live device of the notebook's own account. */
68 export function createNotebook: 
78 export function listMine: 
83 export function listPublic: 
89 export function getNotebook: 
94 export function updateNotebook: 
102  function getVersion: 
108 export function pushVersion: 
145 export function readVersion: 
148  function content: 
157 export const readSource: 
158 export const readRun: 
159 export function deleteVersion: 
165 export function deleteNotebook: 
173 export function purgeOrphans: 
188 export function reportNotebook: 
201  function operator: 
205  function setTakenDown: 
211 export const takeDownNotebook: 
212 export const restoreNotebook: 

## darkpyonix/hub/worker/src/pages.ts (98)
H: The few HTML pages the Worker renders itself. No user input is placed in them unescaped.
3  function escape: 
20 export function sharePlaceholderPage: `/s/{share_id}` before the ash viewer is deployed. */
41 export function linkPage: `/link?code=XXXX-XXXX`: approve or deny a device for the signed-in account. */
63  function show: 
64  function lookup: 
80  function decide: 

## darkpyonix/hub/worker/src/pkarr.ts (242)
H: iroh's pkarr relay protocol (SPEC FR-H2), compatible with iroh-dns 1.3 `SignedPacket`: stored packet = public key (32) || signature (64) || timestamp_us big-endian (8) || DNS packet relay payload = everything after the p…
13 export const MAX_DNS_PACKET: 
22 export function z32Encode: z-base-32 encoding, most significant bit first, no padding. 32 bytes give 52 characters. */
40 export function z32DecodeKey: Decodes a 52-character z-base-32 public key to 32 bytes, or null. */
62 export interface SignedPayload: 
69 export function splitRelayPayload: Splits a relay payload. Null if its size is out of range. */
81 export function signable: 
89 export function verifyRelayPayload: Verifies a relay payload for `publicKey`: size, signature, and that the DNS packet parses (iroh-dns …
105 export interface DnsAnswer: 
149  function readName: Reads a (possibly compressed) domain name starting at `start`; returns it and the end offset. */
179 export function parseDnsAnswers: Parses the answer section of a DNS message. Null if the message is malformed. */
221 export interface EndpointAddresses: 
230 export function endpointAddresses: iroh's `_iroh` TXT attributes (iroh-dns `EndpointInfo::from_pkarr_signed_packet`). Like iroh, a valu…

## darkpyonix/hub/worker/src/relay.ts (68)
H: What the relay host asks the hub (SPEC FR-H3). The iroh relay itself does not run on Workers; the relay host calls these with the shared secret during the relay handshake. How long the relay may cache an "allow" for a re…
9 export const ADMIT_CACHE_SECS: How long the relay may cache an "allow" for a registered device before asking again. */
11  function requireRelay: 
20 export function admit: `POST /internal/relay/admit` `{"endpoint_id": hex, "token": string|null}` */
48 export function presence: `POST /internal/relay/presence` `{"endpoint_id": hex, "online": bool}` */

## darkpyonix/hub/worker/src/shares.ts (115)
H: Share links and the ash viewer (SPEC FR-H4). How long a guest relay pass is valid.
10 export const RELAY_PASS_TTL_SECS: How long a guest relay pass is valid. */
17  function shareHost: 
28 export function publishShare: `POST /shares` `{"share_id": "s_..."}` with the hosting device's token. */
50 export function resolveShare: `GET /shares/{share_id}`: public. Returns the host and a fresh guest relay pass. */
75 export function unpublishShare: `DELETE /shares/{share_id}`: the hosting device, or the account (session / main server). */
88  function ashIndex: 
98 export function sharePage: `GET /s/{share_id}`: the ash viewer page (the token after `#` never reaches us). */
111 export function ashPage: `GET /ash/` when the request reaches the Worker (assets normally answer first). */

## darkpyonix/hub/worker/src/util.ts (134)
H: Encoding, hashing, tokens and input validation.
5 export function utf8: 
9 export function concatBytes: 
19 export function toHex: 
26 export function fromHex: Lowercase hex of exactly `len` characters, or null. */
34 export function base64url: 
40 export function fromBase64url: 
53 export function randomBytes: 
57 export function randomHex: 
62 export function newToken: A bearer token: prefix + 32 random bytes in base64url. */
66 export function sha256: 
72 export function hashToken: Tokens and session ids are stored as the hex SHA-256 of the string. */
77 export function ctEq: Constant-time string comparison (length is not hidden). */
86 export function nowSecs: 
92 export const ENDPOINT_ID_RE: 
93 export const SHARE_ID_RE: 
94 export const ACME_VALUE_RE: 
98 export const RESERVED_NAMES: Labels that stay with the hub itself (SPEC FR-H5). */
103 export function isValidName: 
109 export const USER_CODE_RE: 
111 export function newUserCode: 
123 export function normalizeUserCode: Accepts `bcdf ghjk`, `BCDFGHJK`, `BCDF-GHJK` and returns the canonical form or null. */
131 export function safeReturnTo: A same-origin path to return to after login: `/x`, never `//host` or `/\host`. */

## darkpyonix/hub/worker/vitest.config.ts (24)
H: Tests run inside workerd (the pool's own Miniflare), with the bindings of wrangler.toml: D1 (migrated per test file by test/apply-migrations.ts) and the rate limiter.

## darkpyonix/kernel/__init__.py (43)
H: DarkPyonix runtime API for notebook files (SPEC FR-F2..F6, FORMAT §4). Standard library only (INTENT §2). Importing this package must stay cheap and must never import any kernel-process module (``_server``, ``_executor``…
14 const _EXPORTS = {'markdown': '_markdown', 'params': '_params', 'binding': '_…
31 def __getattr__: 
42 def __dir__: 

## darkpyonix/kernel/__main__.py (263)
H: The DarkPyonix kernel process (INTENT D1–D3, D12; PROTOCOL §2–§3). Started as ``python -m darkpyonix`` (or by ``_launcher.bootstrap_command``). Owns the per-file lock, the shared document (SPEC §10a, ``_collab.DocumentSt…
34 const FINISHED = ('ok', 'error', 'interrupted', 'cancelled', 'crashed')
37 def _log: 
45 class Kernel: 
  46 .__init__: 
  71 .info: 
  88 .emit: 
  164 .handlers: 
  181 .request_shutdown: 
  189 .run: 
252 def main: 

## darkpyonix/kernel/_binding.py (134)
H: ``@darkpyonix.binding`` (SPEC FR-F4, FORMAT §3.4, issue #6). The decorator re-evaluates the decorated ``def`` / ``class`` from its source in a snapshot of the defining module's globals taken at definition time. Imports a…
24 const _FUTURE_FLAGS = 0
29 class _Unresolved: 
33 def _resolve: Resolve a dotted-name decorator expression without evaluating anything else.
51 def _first_line: 
55 def _find_node: The def/class named ``name`` whose span (decorators included) contains ``lineno``.
69 def _source_for: (filename, all lines, node) for the decorated definition, or None.
89 def binding: Re-evaluate ``obj`` (a def or class) so it cannot see later ``[code]`` variables.

## darkpyonix/kernel/_capture.py (510)
H: Output capture for the executor (SPEC FR-X5, NFR-K3, INTENT D12). Standard library only; Python 3.8+. - ``OutputRouter`` owns one daemon thread. Producers (the main thread, user threads, fd readers) only append to a dequ…
29 const _POLL = 0.02
30 const _IDLE_POLL = 0.1
33 class OutputRouter: Routes outputs of the active cell to its record and to ``output`` events.
  36 .__init__: 
  52 .add_flushable: 
  55 .remove_flushable: 
  61 .flush_streams: Push buffered stream text into the router queue (any thread).
  71 .start: 
  77 .stop: 
  84 .set_passthrough: 
  96 .write_stream: 
  102 .write_display: 
  111 .write_output: Any nbformat 4 output (used for ``error``).
  116 .clear: 
  120 .begin_cell: Route outputs to ``cell`` of ``run``; events name it by ``index`` and by the sha…
  126 .end_cell: 
  132 .flush: Block until everything queued so far is recorded and emitted.
269 class _RouterSink: The raw end under ``StreamCapture``: whole buffers of bytes, decoded once.
  278 .__init__: 
  286 .writable: 
  289 .readable: 
  292 .seekable: 
  295 .isatty: 
  298 .close: 
  302 .fileno: 
  309 .write: 
324 def StreamCapture: A replacement for ``sys.stdout`` / ``sys.stderr`` that feeds ``router`` (FR-X5, NFR-K3). The result is a plain…
346 class FdCapture: Redirect file descriptors 1 and 2 to pipes read by daemon threads (FR-X5). POSIX and Windows both have ``os.pi…
  354 .__init__: 
  365 .start: 
  393 .original_fd: 
  396 .original_text: A line-buffered text stream on the saved (real) descriptor.
  457 .sync: Wait until everything written to fds 1 and 2 so far has reached the router.
  490 .stop: 

## darkpyonix/kernel/_client.py (215)
H: A blocking DKP/1 client for one kernel (PROTOCOL §3). Used by the CLI, the manager and tests. A background reader thread demultiplexes responses (by request id) and events (into a queue), so ``request`` is safe to call f…
20 const _CLOSED = object()
23 class _Pending: 
  26 .__init__: 
31 class KernelClient: 
  32 .__init__: 
  51 .connect: 
  103 .close: 
  126 .request: 
  156 .subscribe: 
  162 .next_event: The next event, or ``None`` on timeout or once the connection is closed.
  173 .events: 

## darkpyonix/kernel/_collab.py (890)
H: The kernel's shared document: cells, locks and presence (SPEC §10a FR-S1..S5, S8; PROTOCOL §4). Standard library only; Python 3.8+. ``DocumentState`` holds the parsed file in memory and serves the PROTOCOL §4 methods. Th…
63 const CAPABILITY_NAMES = ('read', 'history', 'execute', 'edit', 'manage')
65 const CURSOR_MIN_INTERVAL = 1.0 / 20
67 const _TYPE_RE = re.compile('^[A-Za-z_][A-Za-z0-9_-]*$')
70 def by_of: ``{client_id, user, nickname}``: the ``by`` / ``started_by`` attribution (PROTOCOL §4).
76 def _iso_after: 
81 def _new_cell_id: 
85 def _int_param: 
94 def _str_param: 
103 def _request: Serve a ``DocumentState`` method under its lock with ``params["request_id"]`` set, so every event the request …
122 class _Entry: One cell of the shared document.
  127 .__init__: 
135 class _Lock: 
  138 .__init__: 
146 class _Presence: 
  150 .__init__: 
162 class DocumentState: 
  163 .__init__: 
  208 .start: 
  215 .stop: Stop the background thread and write any pending edit.
  224 .flush: Write pending edits now (e.g. before a run reads the file).
  253 .tick: Do whatever is due at ``clock()``: file poll, debounced save, coalesced cursors,…
  276 .handlers: 
  289 .indices_for: Current indices of ``cell_ids`` (for ``run`` with ``cell_ids``); ``not_found`` i…
  300 .cell_ids: 
  411 .snapshot: 
  423 .cell_create: 
  499 .cell_update: 
  526 .cell_delete: 
  553 .cell_move: 
  575 .lock: 
  592 .unlock: 
  666 .presence_update: 
  706 .presence_leave: 

## darkpyonix/kernel/_command.py (203)
H: ``darkpyonix.run_command`` (SPEC FR-F6) and the package helpers ``uv`` / ``pip``. Output is read by helper threads but written to ``sys.stdout`` / ``sys.stderr`` from the calling thread, one line at a time, so a kernel t…
19 const INTERRUPT_GRACE = 5.0
20 const _POSIX = os.name == 'posix'
23 def _reader: 
33 def _write: 
44 def _signal_group: 
58 def _pump: Write queued lines; returns the number of pipes still open.
70 def run_command: Run ``cmd`` in a subprocess, streaming its output line by line; return the exit code. ``env`` entries are adde…
130 def _drain_until_exit: 
149 def _installed: 
154 class _Pip: ``darkpyonix.pip``: pip for the interpreter running this code.
  157 .__init__: 
  167 .install: 
  174 .uninstall: 
186 class _Uv: ``darkpyonix.uv``: uv (falling back to pip) for the interpreter running this code.
  189 .__init__: 
  192 .add: 
  195 .remove: 

## darkpyonix/kernel/_discovery.py (327)
H: Kernel discovery over loopback UDP multicast, and nothing else (PROTOCOL §2, INTENT D4). FR-D1: a query reaches every kernel of the same user; each answers with an ``announce`` that echoes the query's nonce. Answers go u…
26 const MAX_DATAGRAM = 65507
27 const _GROUP_ADDR = (DISCOVERY_GROUP, DISCOVERY_PORT)
30 def _log: 
38 def _encode: 
42 def _decode: 
52 def _set_send_options: 
58 def open_group_socket: A socket that receives datagrams sent to the discovery group on loopback (§2.1).
85 def open_send_socket: An ephemeral loopback socket for sending to the group and receiving unicast replies.
97 class DiscoveryResponder: The kernel side of PROTOCOL §2: answer queries, announce, say bye.
  100 .__init__: 
  113 .start: 
  126 .stop: 
  139 .announce_now: Multicast the current body (start, status change).
222 class DiscoveryClient: The manager side of PROTOCOL §2: query the group, optionally listen for announces.
  225 .__init__: 
  231 .query: Send one query and collect announces until ``timeout``. Returns early once ``exp…
  276 .listen: Call ``callback(message)`` for each announce or bye of this user, in a daemon th…
  308 .close: 
318 def discover: Live kernels of this user, found by one multicast query (FR-D1).

## darkpyonix/kernel/_display.py (132)
H: Rich display: IPython-style ``_repr_*_`` methods to nbformat MIME bundles (SPEC FR-X5). Standard library only; Python 3.8+.
14 const TEXT_PLAIN_LIMIT = 100000
17 const _REPR_METHODS = (('_repr_html_', 'text/html', 'text'), ('_repr_markdown_', '…
28 def _split: ``_repr_*_`` may return ``value`` or ``(value, metadata)``.
35 def _normalize: The JSON value for one MIME type, or None when ``value`` is unusable.
58 def _plain: 
65 def format_bundle: ``(data, metadata)`` for ``obj``. ``text/plain`` is always present. A failing ``_repr_*_`` method is skipped; …
110 def _lookup: A bound ``_repr_*_`` method defined on the type (not a ``__getattr__`` fabrication).
125 def display: Show each object as a ``display_data`` output; ``print(repr(obj))`` outside a kernel.

## darkpyonix/kernel/_document.py (123)
H: A notebook file with its latest outputs mapped onto its cells (SPEC FR-R4). Standard library only; Python 3.8+. Used by the manager's ``/document`` endpoints; reads ``__runs__/`` directly, so it works whether or not the …
23 const MAX_RUNS_SCANNED = 20
26 def _records: 
42 def _map_run: Match records to the cells at positions ``pending``: id → source_sha256 → index. The index fallback skips a re…
76 def build_document: The OpenAPI ``Document`` for ``path``. ``viewer_outputs=False`` omits outputs (viewer1).

## darkpyonix/kernel/_events.py (77)
H: The kernel's event log: a numbered ring of recent events (PROTOCOL §3.4, PR-3). Standard library only; Python 3.8+.
17 class EventLog: Thread-safe event ring bounded by count and approximate encoded size. ``append`` numbers events from 1 and cal…
  25 .__init__: 
  36 .seq: 
  39 .append: 
  58 .since: Events with ``seq`` greater than ``seq``, and ``oldest_seq`` if some were droppe…
  68 .add_listener: 
  72 .remove_listener: 

## darkpyonix/kernel/_executor.py (714)
H: The executor: runs notebook cells on the main thread (SPEC FR-K5..K8, FR-X1..X6, FR-F5). Standard library only; Python 3.8+. Threading model (INTENT D12): - ``run_forever()`` runs on the main thread and executes one run …
42 const _KERNEL_DIR = os.path.dirname(os.path.abspath(__file__))
43 const _FUTURE_MASK = 0
48 const INJECTED_NAMES = frozenset(('__runs__', 'darkpyonix', '__builtins__', '__name…
51 const _REPR = reprlib.Repr()
54 const REPR_LIMIT = 200
57 def _type_name: 
66 class Executor: Executes runs of one notebook file in one namespace.
  69 .__init__: 
  119 .emit: 
  128 .submit: PROTOCOL §3.3 ``run``.
  206 .cancel: Cancel a queued run (a running run is stopped with ``interrupt``).
  219 .interrupt: Raise ``KeyboardInterrupt`` in the running cell (FR-X4). The queue is kept. ``cl…
  245 .restart: Soft: fresh namespace, execution count 0 (FR-K7). Hard: stop the loop so that th…
  262 .shutdown: Interrupt the running cell, let its run finish as interrupted, end ``run_forever…
  271 .status: 
  286 .namespace: PROTOCOL §3.6. While busy only ``name`` and ``type``: no user code runs here.
  377 .run_forever: Execute runs until ``shutdown()``. Must be called on the main thread.

## darkpyonix/kernel/_figures.py (80)
H: matplotlib figures as ``image/png`` display_data (SPEC FR-X6). Standard library only; Python 3.8+. The kernel never imports matplotlib. ``install()`` puts a meta path finder in front of the import system that watches for…
17 const BACKEND_MODULE = 'darkpyonix._mplbackend'
18 const BACKEND = 'module://' + BACKEND_MODULE
21 def _backend_unset: True when neither MPLBACKEND, matplotlibrc nor ``matplotlib.use()`` chose a backend.
34 class _PyplotHook: A ``sys.meta_path`` finder that chooses the kernel backend as pyplot is imported.
  37 .find_spec: 
  48 .find_module: 
55 def install: 
62 def uninstall: 
72 def flush: Show and close the figures still open at the end of a cell, if pyplot uses our backend.

## darkpyonix/kernel/_home.py (159)
H: The DarkPyonix runtime home (``DARKPYONIX_HOME``, default ``~/.darkpyonix``). PROTOCOL §1 lists what lives here. Standard library only; Python 3.8+.
16 def home: Return the runtime home, honouring ``DARKPYONIX_HOME``.
22 def subdir: Return ``<home>/<name>``, creating it (and the home) with mode 0700 if missing.
29 def _package_dir: The real folder of this package (symlinks resolved, so a link named ``darkpyonix`` to ``kernel`` still counts …
35 def source_root: Return a directory that contains this package under the name ``darkpyonix``. In a source checkout the package …
73 def expose_package: Make ``import darkpyonix`` work in child interpreters of this process (FR-K1). ``multiprocessing`` spawn child…
90 def kernels_dir: 
94 def locks_dir: 
98 def managers_dir: 
102 def write_private: Atomically write ``data`` to ``path`` with mode 0600.
114 def write_public: Atomically write ``data`` to ``path`` with mode 0644 (no secrets).
125 def sockets_dir: 
130 const _MAX_SOCKET_PATH = 100
133 def control_path: Where the kernel's control socket lives (PROTOCOL §3.1): ``<home>/sockets/<kernel_id>.sock``. A runtime home w…
151 def account: The OS account that PROTOCOL §2.2 hashes: ``uid:<n>`` (POSIX) or ``sid:<S-1-5-...>``.
157 def user_tag: ``hex(SHA-256(account))[:16]`` (PROTOCOL §2.2). Derived from the OS account, no key file.

## darkpyonix/kernel/_host.py (38)
H: Defensive access to the kernel host context (``darkpyonix._hostctx``). When the module is missing or reports ``in_kernel() == False`` the runtime API behaves as it does under plain ``python file.py`` (SPEC FR-F5).
11 def hostctx: The host context module, or None when it cannot be imported.
23 def active: The host context module when a kernel is executing code here, else None.
34 def current_params: 

## darkpyonix/kernel/_hostctx.py (85)
H: The bridge between a running kernel and the runtime API (``darkpyonix.*``). Standard library only; Python 3.8+. The executor sets these module globals while it runs; the runtime API reads them. Outside a kernel every fun…
16 def in_kernel: True while code runs inside a DarkPyonix kernel.
21 def current_params: The ``params`` of the run being executed (a copy); ``{}`` outside a kernel.
26 def emit_display: Emit a ``display_data`` output for the cell that is running. ``bundle`` is either a MIME bundle (``{"text/plai…
49 def _install: 
55 def _uninstall: 
62 def _set_params: 
69 def set_active: Host the runtime API without an executor (tests, embedders). ``emit(bundle, silent)`` receives the bundle exac…
83 def clear: Leave the hosted state; the runtime API behaves as under plain ``python file.py``.

## darkpyonix/kernel/_launcher.py (150)
H: Start a kernel for a file in any interpreter, detached from the caller (FR-K1, FR-K4). The interpreter does not need DarkPyonix installed: the bootstrap loads the ``darkpyonix`` package from this file's directory under t…
20 const PACKAGE_DIR = os.path.dirname(os.path.abspath(__file__))
22 const BOOTSTRAP_PRELUDE = "import sys, importlib.util as u; d = %r; s = u.spec_from_fi…
27 const BOOTSTRAP = BOOTSTRAP_PRELUDE + 'from darkpyonix.__main__ import main; s…
29 const READY_STATUSES = ('idle', 'busy')
35 def bootstrap_command: ``[python, "-c", BOOTSTRAP, "--file", <canonical path>, *extra_args]``.
40 def log_path: 
44 def launch: Start a detached kernel for ``path`` and return its pid without waiting.
79 def _reap: If ``pid`` is a kernel we launched and it has exited, collect it.
86 def pid_alive: Whether a process with ``pid`` exists on this machine (zombies count as dead).
109 def _pid_alive_windows: 
130 def wait_for_announce: Poll discovery until ``kernel_id`` reports ``idle`` or ``busy``; None on timeout. With ``pid``, only an entry …

## darkpyonix/kernel/_lock.py (70)
H: One live kernel per file per machine (FR-K3): a non-blocking exclusive OS lock. Standard library only; Python 3.8+. The lock lives on an open file descriptor, so the OS releases it when the process dies for any reason, i…
14 def lock_path: ``<home>/locks/<kernel_id>.lock`` (PROTOCOL §1).
19 class FileLock: Non-blocking exclusive lock on ``path``; the descriptor stays open while held.
  22 .__init__: 
  27 .held: 
  30 .acquire: Take the lock without blocking. Return False if another process holds it.
  49 .release: 

## darkpyonix/kernel/_markdown.py (29)
H: ``darkpyonix.markdown`` (SPEC FR-F2, FORMAT §3.2).
12 def markdown: Render ``text`` as Markdown in a kernel; do nothing under plain ``python file.py``. ``silent=True`` shows the …

## darkpyonix/kernel/_misc.py (66)
H: ``darkpyonix.display``, ``run_parallel`` and the reserved interop calls (FORMAT §4).
9 def display: Show objects as ``display_data`` in a kernel; ``print(repr(obj))`` under plain python.
28 def run_parallel: Run coroutines / awaitables concurrently; return their results in argument order. Values that are not awaitabl…
55 def _reserved: 

## darkpyonix/kernel/_model.py (86)
H: Run and cell records shared by the executor, the run store and the control channel. Standard library only; Python 3.8+. A ``Run`` is one run request (SPEC FR-X1); its notebook form is the run log of FR-R1.
10 const RUN_STATUSES = ('queued', 'running', 'ok', 'error', 'interrupted', 'cancell…
11 const CELL_STATUSES = ('ok', 'error', 'interrupted')
14 class RunRequest: 
  17 .__init__: 
35 class CellRecord: 
  39 .__init__: 
56 class Run: 
  61 .__init__: 
  79 .summary: The ``RunSummary`` of manager.openapi.yaml (``path`` is added by the run store).

## darkpyonix/kernel/_mplbackend.py (53)
H: The kernel's matplotlib backend: figures become ``image/png`` display_data (SPEC FR-X6). matplotlib loads this module itself, as ``module://darkpyonix._mplbackend``, when user code imports pyplot inside a kernel (see ``f…
27 def draw_if_interactive: Interactive mode draws nothing here; figures are shown by show() or at cell end.
31 def _bundle: 
39 def flush_figures: Display every open figure as image/png and close it.
52 def show: 

## darkpyonix/kernel/_params.py (201)
H: ``darkpyonix.params`` (SPEC FR-F3, FORMAT §3.3). A value is resolved in this order: the run request's ``params`` (in a kernel) -> the command line ``--name value`` / ``--name=value`` -> ``default``. With ``choices`` an `…
14 const _MISSING = object()
15 const _TRUE = {'true', '1', 'yes', 'y', 'on'}
16 const _FALSE = {'false', '0', 'no', 'n', 'off'}
19 def _type_name: 
23 def _parse_bool: 
36 def _convert: 
64 def _argv_value: The value of ``--name`` in ``argv`` (last one wins), or _MISSING. Never consumes argv.
89 def _close: 
93 def _in_choices: 
104 class Params: The ``darkpyonix.params`` object: declare parameters and read their values.
  107 .__init__: 
  111 .get: Declare parameter ``name`` and return its resolved value.
  189 .all: Resolved values of every parameter declared so far, in declaration order.
  193 .spec: Descriptions of declared parameters (for clients' parameter forms).

## darkpyonix/kernel/_peer.py (57)
H: Who is on the other end of a local socket (PROTOCOL §3.2, FR-A1). The control channel lets only the same OS account through, judged by what the OS reports about the peer process, not by a secret. Standard library only; P…
17 const _LOCAL_PEERCRED = getattr(socket, 'LOCAL_PEERCRED', 1)
18 const _XUCRED_SIZE = 76
21 def peer_uid: The effective UID of the process that connected ``sock`` (an ``AF_UNIX`` socket).
38 def same_account: True only if the OS says the peer runs as this process's account. Any failure is False.
46 def account: ``uid:<n>`` on POSIX, ``sid:<S-1-5-...>`` on Windows (PROTOCOL §2.2).
53 def _windows_sid: 

## darkpyonix/kernel/_protocol.py (198)
H: DKP/1 primitives shared by the kernel, its clients and the manager (docs/PROTOCOL.md). Standard library only; Python 3.8+. Everything that both ends of the wire must agree on lives here so that the pieces can be built in…
20 const DKP_VERSION = 1
21 const KERNEL_VERSION = __version__
24 const DISCOVERY_GROUP = '239.255.68.80'
25 const DISCOVERY_PORT = 46880
26 const DISCOVERY_INTERFACE = '127.0.0.1'
27 const ANNOUNCE_INTERVAL = 5.0
28 const STALE_AFTER = 15.0
29 const QUERY_TIMEOUT = 0.2
32 const MAX_FRAME = 64 * 1024 * 1024
33 const HANDSHAKE_TIMEOUT = 5.0
36 const EVENT_RING_MAX_EVENTS = 10000
37 const EVENT_RING_MAX_BYTES = 16 * 1024 * 1024
38 const STREAM_COALESCE_SECONDS = 0.05
41 const EXIT_ALREADY_RUNNING = 3
43 const ERROR_CODES = ('bad_request', 'unknown_method', 'busy', 'not_found', 'fram…
49 const KERNEL_STATUSES = ('starting', 'idle', 'busy', 'stopping')
52 class DKPError: An error that travels as ``{"ok": false, "error": {...}}`` (PROTOCOL §3.3).
  55 .__init__: 
  61 .to_dict: 
68 class FrameTooLarge: 
  69 .__init__: 
75 def canonical_path: ``realpath(abspath(path))``, plus ``normcase`` on Windows.
83 def kernel_id_for: ``"k_" + hex(SHA-256(canonical path as UTF-8))[:20]`` (FR-K2).
91 def encode: Serialize one frame. ``dkp`` is added if missing.
101 def _recv_exact: 
113 def recv_frame: Blocking read of one frame from ``sock``.
124 class FrameDecoder: Incremental decoder for non-blocking sockets: ``feed(bytes)`` → complete frames.
  127 .__init__: 
  130 .feed: 
150 def now_iso: UTC timestamp with milliseconds and a ``Z`` suffix.
160 def run_start_iso: ``now_iso()`` for a run start, strictly increasing within the process (FR-R3). Runs sort by ``started_at`` and…
179 def new_run_id: ``YYYYMMDD-HHMMSS-<4 hex>`` in UTC (FR-R1), unique within the process. The suffix is random, so 30 runs in one…

## darkpyonix/kernel/_runs.py (748)
H: Run logs beside the notebook file and the ``__runs__`` magic (SPEC FR-R1, FR-R2, FR-R3, FR-R5). Standard library only; Python 3.8+. Layout (INTENT D8):: <dir of file>/__runs__/<file name>/<run_id>.ipynb one run = one nbf…
34 const RUNS_DIRNAME = '__runs__'
35 const INDEX_NAME = 'index.json'
39 const WRITE_INTERVAL = 0.8
43 const STALENESS_BUDGET = 0.85
44 const MIN_WRITE_INTERVAL = 0.25
45 const DEFAULT_OUTPUT_LIMIT = 16 * 1024 * 1024
46 const OUTPUT_LIMIT_ENV = 'DARKPYONIX_RUN_OUTPUT_LIMIT'
47 const UNFINISHED = ('queued', 'running')
49 const _RUN_ID_RE = re.compile('^[0-9]{8}-[0-9]{6}-[0-9a-f]{4}$')
50 const _TMP_RE = re.compile('^.+\\.[0-9a-f]{8}\\.tmp$')
53 def next_write_interval: Choose the next snapshot interval from the previous write duration.
58 def runs_dir_for: ``<dir of file>/__runs__/<file name>/`` (INTENT D8). Not created.
64 def output_limit: ``DARKPYONIX_RUN_OUTPUT_LIMIT`` in bytes (default 16 MiB).
79 def _cell_id: A valid nbformat 4.5 cell id ([a-zA-Z0-9-_]{1,64}), deterministic from run_id+index.
90 def _summary: RunSummary (without ``path``) from a run's ``metadata.darkpyonix``.
102 def _parse_iso: 
113 def _duration: 
120 def _iso_from_epoch: 
126 def _spill: Keep at most ``limit`` UTF-8 bytes of stream text; return (outputs, excess text). Outputs are already copies. …
175 def _text: 
180 def sidecar_name: ``<run_id>.cell<index>.log`` (FR-R5).
185 def _render: The run log notebook, and per cell index the stream text spilled past ``limit``.
233 def to_notebook: The nbformat 4.5 form of ``run`` (FR-R1), outputs in full (no FR-R5 spill).
240 def _write_atomic: 
259 def _dumps: 
263 def _read_json: 
268 def _sort_key: 
274 class RunStore: Run logs of one notebook file, written by the kernel that owns the file.
  277 .__init__: 
  293 .path_of: 
  301 .begin: Write the log now and add the run to index.json.
  309 .update: Mark ``run`` changed. The log is rewritten within ``WRITE_INTERVAL``, never more…
  319 .finish: Write the final log now and update index.json.
  330 .flush: Write every pending update now (e.g. before the kernel exits).
  439 .list: RunSummary dicts newest first, each with the absolute ``path`` of its log.
  450 .current_run: 
  459 .get: The run log notebook for a run id, ``"latest"`` (newest finished) or ``"current"…
  487 .recover_crashed: Mark logs left ``running``/``queued`` by a dead kernel as ``crashed``. Runs this…
  531 .magic: 
535 def to_notebook_meta: The run fields of ``metadata.darkpyonix`` without rendering cells.
548 def _view: ``value`` with every nested dict made a ``LogDict`` and every list a ``LogList``.
557 def _plain: A deep copy of ``value`` made of plain ``dict`` and ``list`` only.
566 class LogList: A JSON list whose dict items allow attribute access (SPEC FR-R3).
  571 .__init__: 
576 class LogDict: A JSON dict that also answers attribute access (SPEC FR-R3). An attribute is looked up as a key first, then as…
  585 .__init__: 
609 class CellLog: One cell of a run log: ``.text``, ``.stderr``, ``.result`` (SPEC FR-R3).
  619 .text: The cell's ``stdout`` stream output, joined in order.
  624 .stderr: The cell's ``stderr`` stream output, joined in order.
  629 .result: ``text/plain`` of the cell's last ``execute_result``, or ``None``.
640 class RunLog: One run log notebook with attribute access (SPEC FR-R3). ``.path`` and ``.notebook`` are not keys, so ``json.d…
  648 .__init__: 
  654 .path: Absolute path of this run's ``.ipynb`` log.
  659 .notebook: The nbformat dict as plain ``dict``/``list`` (a copy).
  663 .cell: The first cell whose id, then title (str), or file cell index (int) is ``key``.
683 class RunsMagic: The ``__runs__`` object in the kernel namespace. Every value is a fresh JSON-serialisable dict/list that also …
  694 .__init__: 
  698 .dir: 
  706 .current: 
  713 .latest: 
  719 .list: 

## darkpyonix/kernel/_server.py (584)
H: The kernel's DKP/1 control channel server (PROTOCOL §3; FR-A1, PR-1..PR-4). One selector thread serves every client with non-blocking sockets. Outgoing bytes are buffered per client, so a slow reader never blocks other c…
33 const MAX_OUTBUF = 64 * 1024 * 1024
34 const MAX_PREAUTH_FRAME = 64 * 1024
35 const RECV_CHUNK = 256 * 1024
36 const HANDLER_WORKERS = 4
37 const MAX_PENDING_FDS = 4
40 class _Conn: 
  45 .__init__: 
62 def _close_fd: 
69 def _recv: Read from ``c``; descriptors sent with ``SCM_RIGHTS`` are queued in ``c.fds``. ``socket.recvmsg`` exists on 3.…
88 def _error_frame: 
92 def _response: 
103 class ControlServer: 
  104 .__init__: 
  127 .address: The Unix socket path or the Windows pipe name; announced as ``control`` (PROTOCO…
  131 .start: 
  176 .stop: 
  221 .client_count: 

## darkpyonix/kernel/_streams.py (469)
H: Streams the manager handed to the kernel (PROTOCOL §3.7; SPEC FR-K9, FR-S9, PR-5, PR-6). The manager authenticates and authorizes, then passes a client connection (SSE, WebSocket, run-wait long poll) with the request byt…
29 const KINDS = ('events', 'ws', 'wait')
30 const MAX_REQUEST = 64 * 1024
31 const MAX_QUEUED = 64 * 1024 * 1024
32 const SEND_TIMEOUT = 10.0
33 const KEEPALIVE = 15.0
34 const PRESENCE_HEARTBEAT = 10.0
35 const POLL = 0.25
36 const MAX_POLLS = 100
37 const RECOMMENDED_DELAY = 5
38 const WS_GUID = b'258EAFA5-E914-47DA-95CA-C5AB0DC85B11'
39 const OUTPUT_EVENTS = ('output', 'output.clear')
40 const FINISHED = ('ok', 'error', 'interrupted', 'cancelled', 'crashed')
41 const _STOP = object()
42 const _CAPABILITY_ORDER = ('read', 'history', 'execute', 'edit', 'manage')
45 def parse_request: ``(method, path, query, headers)`` of the request head the manager already read.
62 class _Stream: 
  63 .__init__: 
  78 .close: Close now; True if this call did it.
96 def _http: 
101 def _json_response: 
107 def _error_response: 
111 const _STATUS = {'bad_request': (400, 'Bad Request'), 'not_found': (404, 'No…
116 def sse_frame: 
124 class StreamHub: Owns every adopted stream of one kernel.
  127 .__init__: 
  138 .adopt: Take ``sock`` and answer ``raw`` on it. Raises ``DKPError`` (the caller closes).
  160 .close_share: 
  165 .count: 
  169 .stop: 

## darkpyonix/kernel/_tokens.py (193)
H: Kernel access tokens (SPEC FR-K10, FR-A5, FR-A6, PR-7; PROTOCOL §6). The kernel owns its access tokens: initial, login and share tokens. It issues, verifies, revokes and persists them; managers only ask through ``tokens.…
25 const CAPABILITIES = ('read', 'history', 'execute', 'edit', 'manage')
26 const KINDS = ('initial', 'login', 'share')
27 const STORE_NAME = 'tokens.json'
30 def _hash: 
34 def _parse_time: 
44 def _capabilities: 
54 class TokenStore: Tokens of one kernel. ``close_streams(share_id) -> int`` closes the streams handed off with that share (set by…
  58 .__init__: 
  87 .handlers: 
  95 .issue: 
  121 .verify: 
  147 .revoke: 
  162 .list: 
  173 .check_file: Delete the store when the notebook file is gone. True if it was.
  185 .start: 
  192 .stop: 

## darkpyonix/kernel/_winpipe.py (516)
H: The Windows named-pipe control channel (PROTOCOL §3.1-§3.2, FR-A1, #80). The kernel serves ``\\.\pipe\darkpyonix-<user_tag>-<kernel_id>``. The pipe's DACL grants access to this account's SID only and refuses remote clien…
19 const PIPE_PREFIX = '\\\\.\\pipe\\'
20 const BUFFER = 64 * 1024
23 def pipe_name: ``\\.\pipe\darkpyonix-<user_tag>-<kernel_id>`` (PROTOCOL §3.1).
28 def is_pipe_name: 
32 def dacl_sddl: Protected DACL with one entry: generic-all for ``sid`` and nobody else (FR-A1).
39 class PipeSocket: The few ``socket`` calls ``KernelClient`` and ``_protocol`` use, over a pipe stream.
  42 .__init__: 
  46 .settimeout: 
  49 .gettimeout: 
  52 .recv: Up to ``n`` bytes; ``b""`` at end of stream; ``socket.timeout`` when the timeout…
  56 .sendall: 
  59 .shutdown: 
  62 .close: 
68 def serve_connection: Check ``stream``'s peer, then bridge it to one end of a socket pair given to ``deliver``. ``ControlServer`` tr…
127 const _ERROR_IO_PENDING = 997
128 const _ERROR_PIPE_CONNECTED = 535
129 const _ERROR_PIPE_BUSY = 231
130 const _ERROR_BROKEN_PIPE = 109
131 const _ERROR_NO_DATA = 232
132 const _ERROR_PIPE_NOT_CONNECTED = 233
133 const _ERROR_OPERATION_ABORTED = 995
134 const _ERROR_ACCESS_DENIED = 5
135 const _WAIT_TIMEOUT = 258
136 const _WAIT_FAILED = 4294967295
137 const _INFINITE = 4294967295
138 const _EOF_ERRORS = (_ERROR_BROKEN_PIPE, _ERROR_NO_DATA, _ERROR_PIPE_NOT_CONNECT…
141 class _Win32: Prototypes of the Win32 calls used here, declared with explicit types for 64-bit handles.
  147 .get: 
  152 .__init__: 
  205 .last_error: 
  208 .sid_of_process: The SID string of the account process ``pid`` runs as. Raises ``OSError``.
239 class OverlappedPipe: One end of a pipe opened for overlapped I/O, so a read never blocks a write on the handle.
  242 .__init__: 
  267 .abort: Cancel every pending call on this end; later calls fail.
  320 .read: 
  342 .write: 
  359 .flush: 
  366 .peer_pid: 
  378 .peer_sid: 
382 def own_sid: 
387 def connect_client: Open the kernel's pipe. The server may only identify us, never act as us (SQOS identification).
407 class PipeListener: Accepts connections on the kernel's pipe, one thread, one fresh instance per connection.
  410 .__init__: 
  443 .start: 
  448 .stop: 
511 def same_server_account: Client side of FR-A1: the process serving ``pipe`` runs as this account.

## darkpyonix/kernel/_wsync.py (453)
H: The 2025 WebSocket synchronization (SPEC FR-S9; PROTOCOL §5). Two parts, both standard library only: * RFC 6455 framing: ``FrameReader`` (client frames in, masked, fragments joined) and the ``*_frame`` encoders (server f…
23 const PING_INTERVAL = 30.0
24 const IDLE_TIMEOUT = 60.0
25 const MAX_MESSAGE = 1024 * 1024
34 class ProtocolError: The peer broke RFC 6455; ``code`` is the close code to answer with.
  37 .__init__: 
42 def frame: One unmasked, final server frame.
54 def text_frame: 
58 def close_frame: 
62 class FrameReader: Incremental parser of client frames. ``feed`` returns complete messages as ``(opcode, payload)``: text/binary …
  66 .__init__: 
  73 .feed: 
130 def _name: 2025 ``user_name``: the user, else the nickname.
137 def _dp: 
143 class _Doc: A snapshot cell with the attributes ``_document._map_run`` reads.
  146 .__init__: 
152 def match_run_cells: Match the records of run ``log`` to the cells of a snapshot (SPEC FR-S9, K2). Uses the FR-R4 rule (``_document…
175 def _blank_sources: Copy of ``event`` without cell code (no ``read``).
186 class WsSync: The messages of one WebSocket connection (PROTOCOL §5).
  189 .__init__: 
  238 .start: ``users_focus_data`` (once, right after the upgrade). Sets the event cursor, so …
  256 .on_event: 
  365 .on_message: 

## darkpyonix/kernel/format/__init__.py (128)
H: Parser for the DarkPyonix notebook format (docs/FORMAT.md §2, SPEC FR-F1). Standard library only; Python 3.8+. Shared by the kernel, the manager and the runtime API. The data model below is the contract other modules bui…
17 const PREAMBLE = 'preamble'
18 const CODE = 'code'
19 const KNOWN_TYPES = ('preamble', 'code', 'markdown', 'argparse', 'binding', 'she…
24 const TYPE_ALIASES = {'concorrunt': 'concurrent'}
27 class Cell: One cell. ``index`` 0 is the preamble (text before the first marker).
  33 .__init__: 
  46 .to_dict: 
54 class NotebookDocument: A parsed notebook file. ``cells[0]`` is always the preamble (possibly empty).
  59 .__init__: 
  67 .preamble: 
  70 .cell: 
74 def parse: Parse notebook source text. Never raises on unknown types or metadata.
100 def load: Read ``path`` as UTF-8 (BOM tolerated) and parse it. The file is decoded without newline translation so that `…
110 def serialize: Inverse of ``parse``: byte-identical for an unmodified document. A cell without a recorded ``header`` (one bui…

## darkpyonix/kernel/format/_parser.py (119)
H: Line-based parser and serializer for the notebook format (docs/FORMAT.md §2, SPEC FR-F1). Standard library only; Python 3.8+. Layout of a parsed file: the text is cut at every marker line. Each cell owns * ``header``: it…
25 const MARKER_RE = re.compile('^# %%(?:[ \\t]+(?P<title>[^\\[\\n]*?))?(?:[ \\t]…
30 const METADATA_RE = re.compile('^# @(?P<key>[A-Za-z_][A-Za-z0-9_.-]*):(?:[ \\t]*…
32 const BOM = '\ufeff'
35 def _split_lines: Split on ``\n`` only, keeping line endings (unlike str.splitlines, which also cuts on form feeds and Unicode s…
45 def _content: 
53 def _metadata_value: 
60 def source_sha256: SHA-256 of a cell body (FORMAT §2.4). ``\r\n`` is normalised to ``\n`` and trailing blank (empty or whitespace…
74 def parse_cells: Return ``(bom, chunks)``; each chunk is ``(marker_match, metadata_items, header, source)``. The first chunk is…
108 def format_header: Build a marker + metadata block for a cell that has no recorded header.

## darkpyonix/manager/crates/darkpyonix/src/args.rs (267)
H: Command line surface of SPEC FR-C2. Parsing only: nothing here touches the network.
16 pub struct Cli: 
26 pub enum Command: 
55 pub struct FileArg: 
60 pub struct OptFileArg: 
65 pub struct RunArgs: 
85 pub struct LogsArgs: 
96 pub struct RestartArgs: 
104 pub struct ShutdownArgs: 
112 pub struct KernelArgs: 
119 pub struct ShareArgs: 
134 pub struct ManagerArgs: 
151 pub struct Cells: 
154 pub fn parse_cells: `1,3` / `2-4` / `0,2-3` → sorted, de-duplicated cell indexes.
178 pub fn parse_param: `k=v`: `v` is JSON when it parses as JSON (`3`, `true`, `[1,2]`, `"x"`), else the raw string.

## darkpyonix/manager/crates/darkpyonix/src/client.rs (132)
H: Minimal client for `docs/api/manager.openapi.yaml`. A failure to show the user: an API `Error` object, or a local problem in the same shape.
10 pub struct CliError: A failure to show the user: an API `Error` object, or a local problem in the same shape.
16  impl CliError: 
35  impl std::fmt::Display for CliError: 
41 pub fn http_client: 
50 pub struct Api: One manager reached over HTTP with its master token.
56  impl Api: 
120  fn transport: 
124  fn api_error: 

## darkpyonix/manager/crates/darkpyonix/src/commands.rs (573)
H: The client commands of SPEC FR-C2, each a few HTTP calls to a manager.
17 pub struct Out: Where text goes, and in which form.
22  impl Out: 
79 pub fn dispatch: Run one client command; returns the process exit code.
89  fn dispatch_inner: 
243  fn ensure_kernel: `POST /api/kernels`: the kernel for `path`, and whether it was launched now (201).
252  fn run: 
300  fn busy: SPEC FR-C2: a second run of a busy file prints the current run and exits 75.
321  fn logs: 
375  fn print_cells: Outputs of a run notebook or a Document, in cell order.
387  fn urlencode: 
397 pub struct CtrlC: SIGINT as a stream of presses. Installing it stops Ctrl-C from killing the CLI.
402  impl CtrlC: 
426 pub enum Target: 
438  fn follow: Follow the events stream. With `interrupt_on_ctrl_c` the first Ctrl-C sends `POST /interrupt` (never…
517  fn handle_event: Print one event; `Some(exit code)` when the followed run finished.

## darkpyonix/manager/crates/darkpyonix/src/discovery.rs (264)
H: Finding a manager (SPEC FR-C1) and naming a file's kernel (PROTOCOL §2.6). Overrides the executable spawned as `<exe> manager --ephemeral` (tests, wrappers).
13 pub const SELF_EXE_ENV:: Overrides the executable spawned as `<exe> manager --ephemeral` (tests, wrappers).
15 pub const SPAWN_TIMEOUT: Duration: How long a freshly spawned manager has to register itself and answer `/health`.
18 pub const URL_ENV:: A manager given by the environment (an agent that already knows its token).
19 pub const TOKEN_ENV:: 
23 pub struct ManagerLine: The one JSON line a manager prints on stdout once it listens (SPEC FR-M3, FR-C1).
31 pub fn home: `DARKPYONIX_HOME`, else `~/.darkpyonix`.
39  fn healthy: 
50 pub fn connect: The manager named by `DARKPYONIX_MANAGER_URL` and `DARKPYONIX_TOKEN`, else a freshly spawned ephemer…
75  fn self_exe: 
84 pub const MANAGER_LOG_LIMIT: u64: Size past which `manager.log` is rotated to `manager.log.1` before the next spawn (SPEC FR-C1).
88 pub fn open_manager_log: Open `manager.log` for appending with mode 0600 (Unix), first rotating it to `<name>.1` (replacing a…
114 pub fn spawn_ephemeral: Re-execute this binary as `darkpyonix manager --ephemeral` in its own session (so the terminal's Ctr…
189 pub fn resolve_file: The canonical path of a notebook file and its kernel id (PROTOCOL §2.6).
196 pub fn kernel_id_for: 

## darkpyonix/manager/crates/darkpyonix/src/main.rs (33)
H: `darkpyonix`: the CLI agents and people call, and `darkpyonix manager` (INTENT D10). Client commands only speak the manager HTTP API (`docs/api/manager.openapi.yaml`); see SPEC FR-C1 (finding or spawning a manager) and F…
17  fn main: 

## darkpyonix/manager/crates/darkpyonix/src/manager.rs (140)
H: `darkpyonix manager`: the manager half of the single binary (INTENT D10, SPEC FR-M3/M4).
6 pub enum Mode: 
15 pub struct ManagerOpts: What `darkpyonix manager ...` asked for, with the defaults of SPEC FR-M3/FR-M4 applied.
24 pub const DEFAULT_IDLE_TIMEOUT: u64: 
26  impl From<: 
48 pub fn run_manager: Serve the manager API until shutdown (INTENT D10, SPEC FR-C1, FR-M3, FR-M4). `dpx_kernel::RealBacken…
88 pub fn startup_line: The one stdout line a manager prints once it listens (SPEC FR-M3, FR-C1).
95 pub fn stderr_notices: Lines a manager writes to stderr. An ephemeral manager's stderr is `manager.log`, so the token appea…

## darkpyonix/manager/crates/darkpyonix/src/render.rs (309)
H: Turning API objects into terminal text: nbformat outputs, tables, exit codes. EX_TEMPFAIL: the file's kernel is busy with another run (SPEC FR-C2). 128 + SIGINT: the run was interrupted, or the user detached with Ctrl-C.…
5 pub const EXIT_OK: i32: 
6 pub const EXIT_ERROR: i32: 
8 pub const EXIT_BUSY: i32: EX_TEMPFAIL: the file's kernel is busy with another run (SPEC FR-C2).
10 pub const EXIT_INTERRUPTED: i32: 128 + SIGINT: the run was interrupted, or the user detached with Ctrl-C.
13 pub fn exit_code_for_run_status: Exit code for a finished run's status (PROTOCOL §3.4 `run.finished`).
22 pub enum Stream: 
28 pub fn multiline: nbformat multiline strings are a string or a list of strings.
37  fn with_newline: 
49 pub fn render_output: How one nbformat 4 output appears in a terminal: `stream` → its text on stdout or stderr as it came;…
91 pub fn strip_ansi: Remove ANSI escape sequences (CSI `ESC [ ... final`, and lone `ESC x`).
111 pub fn table: A left-aligned text table; the last column is not padded.
139 pub fn str_field<: 
144 pub fn kernels_table: `ps` rows from OpenAPI `Kernel` objects.
164 pub fn variables_table: `vars` rows from OpenAPI `Variable` objects.
187 pub fn kernel_details: `status FILE` lines from an OpenAPI `Kernel` object.
214 pub fn run_summary_line: One line describing a `RunSummary` (used for the busy hint).

## darkpyonix/manager/crates/darkpyonix/src/sse.rs (105)
H: Incremental Server-Sent Events parser for the manager's events stream.
6 pub struct SseEvent: 
16 pub struct SseParser: 
20  impl SseParser: 
35  fn find_blank_line: 
48  fn parse_block: 

## darkpyonix/manager/crates/dpx-core/src/lib.rs (146)
H: Shared contract between the manager crates. `dpx-kernel` implements [`KernelBackend`] against real kernels (DKP/1, discovery, launching the embedded stdlib-Python kernel). `dpx-server` serves `docs/api/manager.openapi.ya…
22 pub struct DpxError: An error that maps onto the OpenAPI `Error` schema and a DKP/1 error code.
31  impl DpxError: 
41 pub type Result<T>: 
45 pub struct KernelInfo: The OpenAPI `Kernel` schema. Built from a discovery announce plus `status` when connected.
71 pub struct StartKernel: `POST /api/kernels` body.
83 pub struct Ensured: Result of [`KernelBackend::ensure`]: whether the kernel was already running (200) or launched (201).
90 pub struct KernelEvent: One PROTOCOL §3.4 event: `{seq, type, time, data}`; `seq` is `None` for `replay_truncated`.
98 pub type EventStream: 
105 pub struct AdoptRequest: A client connection the manager hands to a kernel (PROTOCOL §3.7, SPEC FR-M6). `request` is the HTTP…
119 pub trait KernelBackend: Send: Everything the HTTP layer needs from kernels.

## darkpyonix/manager/crates/dpx-kernel/build.rs (73)
H: Embeds the stdlib-Python kernel sources (the `darkpyonix/kernel` folder, which is the `darkpyonix` package) into the binary (INTENT D3, D10). Generates `$OUT_DIR/embedded.rs` with the file table and a content hash that n…
10  fn collect: 
36  fn main: 

## darkpyonix/manager/crates/dpx-kernel/src/discovery.rs (318)
H: Manager side of PROTOCOL §2: loopback UDP multicast query/announce/bye (FR-D1). There is no registry file and no fallback (FR-D2, INTENT D4). A background listener keeps a live cache of unsolicited announces and byes so …
20 pub type Entry: An announce body (PROTOCOL §2.3) as a JSON object.
22 pub fn entry_pid: 
30 pub const GROUP: Ipv4Addr: 
31 pub const PORT: u16: 
32 pub const INTERFACE: Ipv4Addr: 
33 pub const QUERY_TIMEOUT: Duration: 
34 pub const STALE_AFTER: Duration: 
37  struct Cached: 
42  fn canonical: 
46 pub struct Discovery: 
57  fn set_send_options: 
65  fn open_group_socket: A socket that receives datagrams sent to the discovery group on loopback (§2.1).
84  fn open_send_socket: An ephemeral loopback socket that sends to the group and receives unicast answers.
92  fn decode: 
99  fn str_field<: 
103  impl Discovery: 
304  fn listen: 

## darkpyonix/manager/crates/dpx-kernel/src/dkp.rs (995)
H: DKP/1 client (PROTOCOL §3): length-prefixed JSON frames, peer-account check, request/response by id and event fan-out. One [`Connection`] per kernel is shared by every request and every event subscriber of the manager (F…
21 pub const MAX_FRAME: usize: 
22 pub const HANDSHAKE_TIMEOUT: Duration: 
23 pub const REQUEST_TIMEOUT: Duration: 
24 pub const RING_MAX_BYTES: usize: 
28 pub struct FanoutConfig: Tunables of the event mirror and fan-out.
35  impl Default for FanoutConfig: 
44  fn unreachable: 
48 pub fn encode: 
63  fn read_frame<R: AsyncRead: Read one frame; returns the parsed object and its payload size.
88  fn open_stream: Open the kernel's Unix socket and ask the OS who serves it (PROTOCOL §3.2, FR-A1): a socket of anoth…
106  fn require_same_account: The kernel's account must be ours; otherwise nothing is sent to it (FR-A1).
120  fn open_stream: Open the kernel's named pipe and ask the OS which account serves it (PROTOCOL §3.2, FR-A1, #80). A p…
157  fn require_same_sid: The kernel's account must be ours (compared as binary SIDs); otherwise nothing is sent to it (FR-A1)…
250 pub fn now_iso: UTC now as `YYYY-MM-DDTHH:MM:SS.mmmZ`.
275  fn truncated_event: 
284  struct Events: 
296  impl Events: 
302  struct Shared: 
310  impl Shared: 
424 pub struct Connection: A control connection to one kernel.
432  impl Connection: 
597  impl Drop for Connection: 
603  struct SubState: 
610  fn next_event: 
641  fn write_loop<W: AsyncWrite: 
658  fn read_loop<R: AsyncRead: 
688 pub fn base64_encode: Standard base64 with padding (PROTOCOL §3.7.1 carries the request bytes this way).
705  fn sendmsg_fd: One byte `F` carrying `fd` in `SCM_RIGHTS` (PROTOCOL §3.7.2).
746 pub fn adopt: Hand a client connection to the kernel on a fresh control connection of its own, so that no other fr…
806 pub fn adopt: 

## darkpyonix/manager/crates/dpx-kernel/src/home.rs (143)
H: The runtime home (`DARKPYONIX_HOME`, default `~/.darkpyonix`), PROTOCOL §1. Mirrors `darkpyonix/kernel/_home.py`. `DARKPYONIX_HOME` or `~/.darkpyonix`, made absolute.
12 pub fn default_home: `DARKPYONIX_HOME` or `~/.darkpyonix`, made absolute.
20  fn home_dir: 
29 pub fn subdir: `<home>/<name>`, created with mode 0700 (and the home) when missing.
35 pub fn create_dir_private: 
46 pub fn random_hex: 
53 pub fn account: The OS account that PROTOCOL §2.2 hashes: `uid:<n>` (POSIX) or `sid:<SID>` (Windows).
141 pub fn user_tag: `hex(SHA-256(account))[:16]` (PROTOCOL §2.2). No key file is read or created.

## darkpyonix/manager/crates/dpx-kernel/src/ident.rs (235)
H: Kernel identity (PROTOCOL §2.6, FR-K2): `kernel_id = "k_" + hex(SHA-256(canonical))[:20]` with `canonical = realpath(abspath(path))` (plus `normcase` on Windows). The functions port Python's `posixpath.abspath`/`realpath…
13 pub fn kernel_id_for: `"k_" + hex(SHA-256(canonical path as UTF-8))[:20]`.
18 pub fn kernel_id_for_canonical: The id of an already canonical path.
24 pub fn canonical_path: `realpath(abspath(path))`, plus `normcase` on Windows. Lossy for non-UTF-8 paths.
29 pub fn abspath: `os.path.abspath`: absolute and lexically normalised (no symlink resolution).

## darkpyonix/manager/crates/dpx-kernel/src/launch.rs (278)
H: The embedded stdlib-Python kernel (INTENT D3, D10): extraction to the runtime home, interpreter selection, detached launch (FR-K1, FR-K4, FR-M2) and the FR-R4 document builder, which stays in Python (`darkpyonix._documen…
18 pub const DOCUMENT_TIMEOUT: Duration: 
20  fn internal: 
25 pub fn runtime_tag: `<crate version>-<content hash>`: the name of the extraction directory.
32 pub fn extract_runtime: Extract the embedded `darkpyonix` package once to `<home>/runtime/<tag>/` and return that directory …
61  fn py_str: A Python string literal for `s` (JSON string syntax is valid Python).
67 pub fn bootstrap: `import sys; sys.path.insert(0, ROOT); from darkpyonix.__main__ import main; ...` (launcher.py `BOOT…
74  fn find_on_path: 
95 pub fn resolve_python: `requested`, else `DARKPYONIX_PYTHON` (via `configured`), else `python3`/`python` on PATH.
107 pub struct Launch<: 
118 pub struct Spawned: A launched kernel process: its pid and a way to stop it while it is still our child.
123  impl Spawned: 
135 pub fn spawn_kernel: Spawn `[python, "-c", BOOTSTRAP, "--file", canonical]` detached from the manager, with output append…
183  fn detach: 
196  fn detach: 
202  fn open_log: 
229 pub fn build_document: FR-R4: run `darkpyonix._document.build_document` in `python`.

## darkpyonix/manager/crates/dpx-kernel/src/lib.rs (484)
H: `RealBackend`: the [`KernelBackend`] of the DarkPyonix manager against real kernels. discovery: loopback UDP multicast only, no registry files (FR-D1, FR-D2); one shared DKP/1 connection per kernel for requests and event…
34 pub const START_TIMEOUT: Duration: FR-M2: how long `ensure` waits for a launched kernel to announce itself.
45 pub struct Config: Backend settings. [`Config::from_env`] reads `DARKPYONIX_HOME` and `DARKPYONIX_PYTHON`; tests build …
54  impl Config: 
75  type Slot: 
77 pub struct RealBackend: 
86  fn internal: 
90  fn not_found: 
95 pub fn kernel_info: Build the OpenAPI `Kernel` from an announce / `status` result.
143  impl RealBackend: 
346  impl KernelBackend for RealBackend: 

## darkpyonix/manager/crates/dpx-kernel/src/process.rs (98)
H: OS process helpers: liveness checks and force-kill by pid. Whether a process with `pid` exists on this machine (owned by anyone). SIGKILL / TerminateProcess. `Ok(false)` when the process does not exist.
4 pub fn pid_alive: Whether a process with `pid` exists on this machine (owned by anyone).
12 pub fn kill_pid: SIGKILL / TerminateProcess. `Ok(false)` when the process does not exist.

## darkpyonix/manager/crates/dpx-server/src/api.rs (1979)
H: Every operation of `docs/api/manager.openapi.yaml` (SPEC FR-M1) with the per-operation required capability of SPEC FR-A3 / FR-S8. Collaborative operations (SPEC §10a, PROTOCOL §4) are thin: the kernel holds the shared do…
29 pub struct AppState: Everything the handlers share.
49  type St: 
82  fn finish: 
92 pub struct RawBody: The raw request body; size-limit rejections keep the Error shape.
94  impl <S: Send: 
105  fn parse_json<T: DeserializeOwned>: JSON body validation: any failure is `400 bad_request` with `data.errors` (never 422).
112  fn query: 
119  fn flag: 
132  fn clamp: `limit`-style integers are lenient (prototype and pytest oracle): an unparsable value falls back to …
138  fn check_id: 
146  fn require: 
156  fn check_user_permission: The 2025 request argument `user_permission` (SPEC FR-A3): when given it names the capability the ope…
168  fn parse_capabilities: `capabilities` of a request body: a non-empty list of the five names, nothing else.
176  fn body_user_permission: `user_permission` from a JSON body that is optional as a whole (`null`, empty or not an object count…
180  fn visible: 
185  fn call: 
190 pub fn kernel_json: The OpenAPI `Kernel` object.
202  fn ok_json: 
213  fn header_str<: A header as UTF-8 (nicknames are device names and need not be ASCII), trimmed, non-empty.
221  fn check_client_id: 
229  fn check_nickname: 
239 pub fn client_value: The PROTOCOL §4 `client` object: `{client_id, nickname, user, avatar?, capabilities}`. The user is t…
274  fn client_ident: Client id and nickname from the headers of an operation that needs a client (edits, locks, presence)…
283  fn required_client: 
290  fn optional_client: `client` for run attribution (`started_by` / `interrupted_by`, FR-S6). The client id is optional her…
297  fn check_cell: `cell_id` path parameter (`^[A-Za-z0-9_.:-]{1,64}$`); anything else names no cell.
306  fn field: `result[name]` of a kernel answer.
315 pub fn get_health: 
319 pub fn get_manager: 
343 pub fn list_kernels: 
355  struct StartKernelBody: 
365 pub fn start_kernel: 
390 pub fn get_kernel: 
403 pub fn shutdown_kernel: 
442 pub fn interrupt_kernel: 
470 pub fn restart_kernel: 
510 pub fn get_namespace: 
530  fn shape_document: 
552 pub fn merge_snapshot: FR-S1: the kernel's live document (`doc.snapshot`: cell ids, versions, locks, conflicts, presence, `…
637 pub fn get_kernel_document: 
661 pub fn get_document: 
684 pub fn list_runs: 
699  enum RunMode: 
706  enum OnBusy: 
713  struct RunBody: 
728 pub fn start_run: 
771  fn check_run_ref: 
780 pub fn run_summary: `RunSummary` from a run log's `metadata.darkpyonix` (FR-R1).
804 pub fn get_run: 
838 pub fn cancel_run: 
861 pub fn stream_events: 
923 pub fn sync_ws: The 2025 WebSocket sync (SPEC FR-S9): the kernel answers the upgrade. Plain TCP connections are hand…
953  fn presence_heartbeat: `presence.update {client}` now and every `presence_heartbeat` until aborted (the event stream closed…
974  fn base64_decode: 
998  fn basic_password: The password of `Authorization: Basic base64(user:password)`; the user name is ignored.
1008  fn ephemeral_forbidden: 
1012  fn password_not_set: 
1018  fn check_pw: Checks `password` against the manager's. PBKDF2 is slow, so it runs off the async threads. Wrong pas…
1041  fn check_basic: `Authorization: Basic` with the manager's password (login and master token reset).
1049 pub fn login: 
1059  struct PasswordBody: 
1063 pub fn set_password: 
1086 pub fn reset_master_token: 
1095 pub fn reset_shared_token: 
1137  struct InitialBody: 
1142  fn issued_token: 
1146 pub fn create_initial_token: 
1167 pub fn kernel_login: 
1187 pub fn verify_token: 
1247  fn kernel_share: A kernel-held share (`tokens.list`) as an OpenAPI `Share`.
1258 pub fn list_shares: 
1285  struct ShareBody: 
1298 pub fn create_share: 
1336 pub fn create_share_token: The 2025 `POST /kernels/{kernel_id}/tokens/share`: the kernel issues and holds the token, so it work…
1390 pub fn revoke_share: 
1424  struct CellEditBody: `CellEdit` plus `after`/`before` (createCell) and `base_version` (updateCell).
1443  fn present<: Deserialize a field that is present (possibly `null`) into `Some(..)`.
1454  fn with_rid: Copy `X-DarkPyonix-Request` into the kernel params as `request_id` (PROTOCOL §4).
1464  fn edit_params: `{client, type?, source?, metadata?}` for `doc.cell.create` / `doc.cell.update`.
1482 pub fn create_cell: 
1512 pub fn update_cell: 
1537 pub fn delete_cell: 
1568 pub fn move_cell: 
1602 pub fn lock_cell: 
1622 pub fn unlock_cell: 
1664  fn is_index: 
1669  fn check_cursor: `Cursor` (`{cell_id, line, column, selection?: [[l, c], [l, c]]}`) or null.
1696 pub fn update_presence: 
1750 pub fn leave_presence: 
1782  fn poll_params: `poll_sequence` (integer >= 1, default 1) and the optional `client_id` of a wait request.
1806 pub fn wait_run: 
1872 pub enum HandoffKind: The three streams a manager hands to the kernel (PROTOCOL §3.7).
1878  impl HandoffKind: 
1889 pub struct HandoffPlan: What the kernel gets besides the request bytes.
1897 pub fn capabilities: FR-A3 capability names of a principal; the kernel only filters by these names (FR-K9).
1903 pub fn plan_handoff: The checks the three stream operations make, without touching the kernel's stream: who may see which…
1962  fn close_share_streams: A share was revoked: the kernel closes the streams it adopted for that share (PROTOCOL §3.7.4).
1973 pub fn not_found: 
1977 pub fn method_not_allowed: 

## darkpyonix/manager/crates/dpx-server/src/auth.rs (678)
H: Manager tokens, password and permissions (SPEC FR-A2, FR-A3, FR-A4, FR-M4). The master token is random per manager process unless configured. Share tokens exist only on a dedicated manager; they are stored as SHA-256 has…
20 pub enum Capability: One of the five capabilities of SPEC FR-A3 (INTENT D18). They have no order.
28  impl Capability: 
53 pub struct Caps: A set of capabilities.
55  impl Caps: 
87 pub const CAPABILITIES:: The five capabilities of SPEC FR-A3, in their canonical order.
91 pub fn parse_token_type: `read+history` to `["read", "history"]`; `None` unless the names are known, unique and in the canoni…
106 pub struct Principal: Who is calling: the master token (all capabilities) or a token bound to one kernel.
118  impl Principal: 
131  fn parse_caps: `read+history` as stored in `manager.db`; `None` unless every name is a capability.
136 pub fn sha256_hex: 
140 pub fn random_hex: 
146  fn ct_eq: 
158  struct HmacSha256: 
163  impl HmacSha256: 
195  fn pbkdf2_sha256: PBKDF2-HMAC-SHA256 with one 32-byte output block (RFC 8018).
211 pub fn hash_password: `pbkdf2-sha256$<iterations>$<salt hex>$<hash hex>`: a salted slow hash (SPEC FR-A4).
218 pub fn check_password: 
234 pub enum DbShare: A token looked up among the manager database shares.
242 pub enum Lookup: How a token looked up against this manager's own records.
253 pub const DEFAULT_FREE_ATTEMPTS: u32: 
254 pub const DEFAULT_DELAY_BASE: Duration: 
260 pub struct AttemptLimiter: Exponential delay on repeated wrong passwords (SPEC FR-A4, #87). One counter per manager, because th…
266  impl AttemptLimiter: 
299 pub enum PasswordCheck: Result of checking a password against the manager's.
307 pub struct Share: A share as listed (`Share` schema). The token itself is never stored.
317 pub struct ShareStore: SQLite share store of a dedicated manager (`<home>/manager.db`).
321  impl ShareStore: 
589  struct Master: The master token: its hash, and its value while this process knows it.
596 pub struct Auth: Token check for every request: master first (constant time), then manager shared tokens, then per-ke…
602  impl Auth: 

## darkpyonix/manager/crates/dpx-server/src/docs.rs (29)
H: The API reference page (`/docs/`) and the two OpenAPI files it renders. These are static files outside the contract (SPEC NFR-M3 exception), embedded in the binary.
10 pub const INDEX_HTML:: 
11 pub const MANAGER_OPENAPI:: 
12 pub const HUB_OPENAPI:: 
14 pub fn redirect: 
18 pub fn index: 
22 pub fn file: 

## darkpyonix/manager/crates/dpx-server/src/error.rs (130)
H: The OpenAPI `Error` / `BusyError` responses and the mapping from [`DpxError`] codes. An HTTP error in the contract's `{"error": {"code", "message", "data"?}}` shape.
10 pub struct ApiError: An HTTP error in the contract's `{"error": {"code", "message", "data"?}}` shape.
17  impl ApiError: 
68  impl From<DpxError> for ApiError: 
104 pub fn error_body: 
114  impl IntoResponse for ApiError: 
130 pub type ApiResult<T>: 

## darkpyonix/manager/crates/dpx-server/src/handoff.rs (487)
H: Long-lived streams go to the kernel (SPEC FR-M6, PROTOCOL §3.7, INTENT D6). The manager authenticates and authorizes, then the kernel answers the client itself: Plain TCP** (HTTP/1.1, no TLS): [`intercept`] peeks the req…
25 pub struct Handoff: Marks a response whose stream the kernel owns: the idle timer does not count it (FR-M3).
31 pub struct Parsed: A stream request parsed from the head bytes of a plain connection.
40  fn find: 
45 pub fn parse_head: `Some` only for `GET` HTTP/1.1 requests without a body on one of the three stream paths.
89 pub fn token_of: The bearer token, or for events and WebSocket (which cannot set headers in a browser) `?token=`.
107 pub fn build_request: The request as the kernel gets it: same path and headers, no token anywhere.
142 pub fn pump_response: 

## darkpyonix/manager/crates/dpx-server/src/lib.rs (618)
H: HTTP layer of the DarkPyonix manager (SPEC FR-M1..M4, FR-A2/A3, INTENT D10). [`serve`] answers every operation of `docs/api/manager.openapi.yaml` on top of any [`KernelBackend`], plus three things outside the contract: t…
55 pub const VERSION:: Manager version reported by `/health`, and `getManager`.
56 pub const DEFAULT_IDLE_TIMEOUT: Duration: 
57 pub const DEFAULT_DEDICATED_PORT: u16: 
58 pub const DEFAULT_SHARE_BASE:: 
62 pub const PRESENCE_HEARTBEAT: Duration: `presence.update` heartbeat while a client's event stream is open (FR-S4; the kernel expires a clien…
68 pub enum Mode: 
73  impl Mode: 
83 pub struct ServerConfig: 
109  impl ServerConfig: 
134 pub fn default_home: `$DARKPYONIX_HOME`, else `~/.darkpyonix`.
145 pub struct Activity: Open requests and streams, and when the last one ended (idle watchdog).
150  impl Activity: 
171 pub struct ActivityGuard: 
173  impl Drop for ActivityGuard: 
181  struct GuardedBody: A response body that keeps its request counted as active until it is fully sent or dropped.
186  impl HttpBody for GuardedBody: 
200  fn track: 
214  fn is_ws_path: `/api/ws/kernels/{id}`: the browser WebSocket API cannot set headers either.
219  fn is_events_path: 
229  fn query_token: 
237  fn is_open_auth_path: Paths that authenticate by themselves, without `Authorization: Bearer` (SPEC FR-A2): the manager log…
253  fn path_kernel_id: The kernel id in `/api/kernels/{id}/...` or `/api/ws/kernels/{id}`.
264  fn kernel_principal: Asks the kernel about a token it may hold (`tokens.verify`, PROTOCOL §6).
290  fn auth_gate: Authenticates before routing, so an anonymous caller always gets 401 (never 400/404).
348  fn proxy_gate: 
355 pub fn router: 
403 pub enum ExitReason: Why [`ServerHandle::wait`] returned.
412 pub struct ShutdownTrigger: Stops the server from anywhere (signal handlers, CLI).
414  impl ShutdownTrigger: 
420 pub struct ServerHandle: 
430  impl ServerHandle: 
443  fn bind: 
461 pub fn serve: Starts the manager's HTTP server and returns once it is accepting connections.
599 pub fn run_until_signal: Serves until SIGINT/SIGTERM (or idle exit) and returns why it stopped.

## darkpyonix/manager/crates/dpx-server/src/proxy.rs (247)
H: Generic reverse proxy routes (INTENT D10: the manager replaces nginx in front of VS Code `serve-web`, ember, ...). Outside the OpenAPI contract, like `/docs/`. Plain HTTP requests are forwarded with a pooled client; HTTP…
21 pub struct ProxyRoute: 
31  impl ProxyRoute: 
47 pub struct ConnInfo: The peer and transport of the accepted connection (set by the accept loop).
52 pub struct Proxy: 
68  fn strip_hop_by_hop: 
84  fn is_upgrade: 
92  fn bad_gateway: 
96  impl Proxy: 

## darkpyonix/manager/crates/dpx-server/src/sse.rs (97)
H: `GET /kernels/{id}/events` as Server-Sent Events (SPEC FR-M1, PROTOCOL §3.4). Frames are `id: <seq>`, `event: <type>`, `data: <json of data>`; `replay_truncated` has no `id` so it never moves the client's `Last-Event-ID`…
19 pub fn frame: 
30 pub struct AbortOnDrop: Aborts a background task (the presence heartbeat of a stream) when dropped.
32  impl Drop for AbortOnDrop: 
38  struct Feed: 
48 pub fn response: 

## darkpyonix/manager/crates/dpx-server/src/tls.rs (76)
H: TLS through rustls (INTENT D10: no nginx). HTTP/2 is negotiated with ALPN. Certificates are served through a swappable resolver ([`CertStore`]) so that ACME (automatic issuance and renewal for a dedicated manager) can re…
18 pub enum TlsConfig: 
28 pub struct CertStore: The current certificate; ACME renewal will call [`CertStore::set`].
32  impl CertStore: 
38  impl ResolvesServerCert for CertStore: 
44  fn io_err: 
48 pub fn certified_key_from_pem: 
61 pub fn acceptor: Builds the acceptor and the certificate store behind it.

## darkpyonix/manager/crates/dpx-server/src/util.rs (164)
H: Small helpers: time stamps, id patterns, private files.
9 pub fn iso: 
14 pub fn now_iso: 
18 pub fn unix_now: 
22 pub fn parse_datetime: 
26  fn hex_lower: 
31 pub fn is_kernel_id: `^k_[0-9a-f]{20}$`
36 pub fn is_share_id: `^s_[0-9a-f]{16}$`
41 pub fn is_run_id: `^[0-9]{8}-[0-9]{6}-[0-9a-f]{4}$`
52 pub fn chmod_private: 
58 pub fn chmod_private: 
61 pub fn ensure_private_dir: Creates `dir` (0700 on Unix) if missing.
76 pub fn write_private_atomic: Writes `data` to `path` atomically (temp file + rename), mode 0600 from the start.
106 pub fn os_user: The OS user running the manager: the user name of master-token clients (FR-S4).
116 pub fn is_client_id: `^[A-Za-z0-9_-]{8,64}$` (OpenAPI `X-DarkPyonix-Client`).
121 pub fn is_cell_id: `^[A-Za-z0-9_.:-]{1,64}$` (OpenAPI `CellId`).
126 pub fn runs_dir_for: `<file dir>/__runs__/<file name>` (OpenAPI `Kernel.runs_dir`).

## packaging/hatch_build.py (50)
H: Wheel build hook (SPEC FR-C3, INTENT D22). Two kinds of wheel come out of the same pyproject.toml: * Platform wheel: ``DARKPYONIX_BINARY`` names a compiled Rust ``darkpyonix`` binary and ``DARKPYONIX_WHEEL_TAG`` the plat…
22 const FALLBACK_SCRIPT = 'packaging/darkpyonix-unsupported'
25 class CustomBuildHook: 
  26 .initialize: 

## packaging/check_versions.py (55)
H: One version, three places (SPEC FR-C3, INTENT D22). The source of truth is ``__version__`` in darkpyonix/kernel/__init__.py. The Cargo workspace version (darkpyonix/manager/Cargo.toml) and, with ``--binary``, the output …
15 const ROOT = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
18 def package_version: 
23 def cargo_version: 
29 def main: 

## packaging/smoke.py (222)
H: Smoke test of an installed darkpyonix wheel (SPEC FR-C3 acceptance criteria 1 to 5). Run it with any Python 3 interpreter, from a directory that is not the checkout:: python -I packaging/smoke.py --venv DIR --version 0.3…
22 const TIMEOUT = 90
25 def say: 
29 const STATE = {'home': None, 'mgr': None}
32 def dump_diagnostics: On failure: the manager's own output and every file under the smoke home (logs).
69 def fail: 
78 def run: 
94 def diagnose_launcher: Failure-only probes: is the venv python a launcher stub whose pid is not the kernel's?
111 def api_get: 
119 def main: 
