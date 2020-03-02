### Configuration Options


##### message `Config` (config/config.proto)

The top-level configuration object. For a simple example, see the [sample JSON in the bookinfo configmap template](https://github.com/istio-ecosystem/authservice/blob/master/bookinfo-example/config/authservice-configmap-template.yaml).

| Field | Description | Type |
| ----- | ----------- | ---- |
| chains | Each incoming http request is matched against the list of filters in the chain, in order, until a matching filter is found. The first matching filter is then applied to the request. After the first match is made, other filters in the chain are ignored. Order of chain declaration is therefore important. At least one `FilterChain` is required in this array. | (slice of) FilterChain |
| listen_address | The IP address for the authservice to listen for incoming requests to process. Required. | string |
| listen_port | The TCP port for the authservice to listen for incoming requests to process. Required. | int32 |
| log_level | The verbosity of logs generated by the authservice. Must be one of `trace`, `debug`, `info', 'error' or 'critical'. Required. | string |
| threads | The number of threads in the thread pool to use for processing. The main thread will be used for accepting connections, before sending them to the thread-pool for processing. The total number of running threads, including the main thread, will be N+1. Required. | uint32 |



##### message `Endpoint` (config/common/config.proto)

A URI definition.

| Field | Description | Type |
| ----- | ----------- | ---- |
| scheme | The scheme, which must be set to `https`. Required. | string |
| hostname | The hostname. Required. | string |
| port | The port number. Required. | int32 |
| path | The path, which must begin with a forward slash (i.e. `/`). Required. | string |



##### message `Filter` (config/config.proto)

A filter configuration.

| Field | Description | Type |
| ----- | ----------- | ---- |
| type | The type of filter. Currently, the only valid type is `oidc`. Required. | oneof |
| oidc | An OpenID Connect filter configuration. | oidc.OIDCConfig |



##### message `FilterChain` (config/config.proto)

A chain of one or more filters that will sequentially process an HTTP request.

| Field | Description | Type |
| ----- | ----------- | ---- |
| name | A user-defined identifier for the processing chain used in log messages. Required. | string |
| match | A rule to determine whether an HTTP request should be processed by the filter chain. If not defined, the filter chain will match every request. Optional. | Match |
| filters | The configuration of one of more filters in the filter chain. When the filter chain matches an incoming request, then this list of filters will be applied to the request in the order that they are declared. All filters are evaluated until one of them returns a non-OK response. If all filters return OK, the envoy proxy is notified that the request may continue. The first filter that returns a non-OK response causes the request to be rejected with the filter's returned status and any remaining filters are skipped. At least one `Filter` is required in this array. | (slice of) Filter |



##### message `LogoutConfig` (config/oidc/config.proto)

When specified, the authservice will destroy the authservice session when a request is made to the configured path.

| Field | Description | Type |
| ----- | ----------- | ---- |
| path | A http request path that the authservice matches against to initiate logout. Whenever a request is made to that path, the authservice will remove the authservice-specific cookies and respond with a redirect to the configured `redirect_to_uri`. Removing the cookies causes the user to be unauthenticated in future requests. If the service application has its own logout controller, then it may be desirable to have its logout controller redirect to this path. If the service application does not need its own logout controller, then the application's logout button/link's href can GET or POST directly to this path. Required. | string |
| redirect_to_uri | A URI specifying the destination to which the authservice will redirect any request made to the logout `path`. For example, it may be desirable to redirect the logged out user to the homepage of the service application, or to the [logout endpoint of the OIDC Provider](https://openid.net/specs/openid-connect-session-1_0.html#RPLogout). As with all redirects, the user's browser will perform a GET to this URI. Required. | string |



##### message `Match` (config/config.proto)

Specifies how a request can be matched to a filter chain.

| Field | Description | Type |
| ----- | ----------- | ---- |
| header | The name of the http header used to match against. Required. | string |
| criteria | The criteria by which to match. Must be one of `prefix` or `equality`. Required. | oneof |
| prefix | The expected prefix. If the actual value of the header starts with this prefix, then it will be considered a match. | string |
| equality | The expected value. If the actual value of the header exactly equals this value, then it will be considered a match. | string |



##### message `OIDCConfig` (config/oidc/config.proto)

The configuration of an OpenID Connect filter that can be used to retrieve identity and access tokens via the standard authorization code grant flow from an OIDC Provider. Retrieved tokens are encrypted and placed in cookies for use in subsequent requests.

| Field | Description | Type |
| ----- | ----------- | ---- |
| authorization | The OIDC Provider's [authorization endpoint](https://openid.net/specs/openid-connect-core-1_0.html#AuthorizationEndpoint). Required. | common.Endpoint |
| token | The OIDC Provider's [token endpoint](https://openid.net/specs/openid-connect-core-1_0.html#TokenEndpoint). Required. | common.Endpoint |
| jwks_config | The OIDC Provider's JWKS configuration used during `id_token` verification. Use either `jwks_uri` or `jwks` (see below). Required. | oneof |
| jwks_uri | *This is currently ignored.* In a future version it will be the URL of the OIDC provider’s public key set to validate signature of the JWT. See [OpenID Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderMetadata). This should match the `jwksUri` value of [Istio Authentication Policy](https://istio.io/docs/tasks/security/authn-policy/). | common.Endpoint |
| jwks | The JSON JWKS response from the OIDC provider’s `jwks_uri` URI which can be found in the OIDC provider's [configuration response](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfigurationResponse). Note that this JSON value must be escaped when embedded in a json configmap (see [example](https://github.com/istio-ecosystem/authservice/blob/master/bookinfo-example/config/authservice-configmap-template.yaml)). | string |
| callback | This value will be used as the `redirect_uri` param of the authorization code grant [Authentication Request](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest). This URL must be one of the Redirection URI values for the Client pre-registered at the OIDC provider. Note: The Istio gateway's VirtualService must be prepared to ensure that this URL will get routed to the service so that the authservice can intercept the request and handle it (see [example](https://github.com/istio-ecosystem/authservice/blob/master/bookinfo-example/config/bookinfo-gateway.yaml)). Required. | common.Endpoint |
| client_id | The OIDC client ID assigned to the filter to be used in the [Authentication Request](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest). Required. | string |
| client_secret | The OIDC client secret assigned to the filter to be used in the [Authentication Request](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest). Required. | string |
| scopes | Optional additional scopes passed to the OIDC Provider in the [Authentication Request](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest). The `openid` scope is always sent to the OIDC Provider, and does not need to be specified here. Required, but an empty array is allowed. | (slice of) string |
| cookie_name_prefix | A unique identifier of the authservice's browser cookies. Can be any string. Only needed when multiple services in the same domain are each protected by their own authservice, in which case each service's authservice should have a unique value to avoid cookie name conflicts. Optional. | string |
| id_token | The configuration for adding ID Tokens as headers to requests forwarded to a service. Required. | TokenConfig |
| access_token | The configuration for adding Access Tokens as headers to requests forwarded to a service. Optional. | TokenConfig |
| logout | When specified, the authservice will destroy the authservice session when a request is made to the configured path. Optional. | LogoutConfig |
| max_absolute_session_timeout | The Authservice associates obtained OIDC tokens with a session ID in a session store. It also stores some temporary information during the login process into the session store, which will be removed when the user finishes the login. This configuration option sets the number of seconds since a user's session with the Authservice has started until that session should expire. When configured to `0`, which is the default value, the session will never timeout based on the time that it was started, but can still timeout due to being idle. When both `max_absolute_session_timeout` and `max_session_idle_timeout` are zero, then sessions will never expire. These settings do not affect how quickly the OIDC tokens contained inside the user's session expire. | uint32 |
| max_session_idle_timeout | The Authservice associates obtained OIDC tokens with a session ID in a session store. It also stores some temporary information during the login process into the session store, which will be removed when the user finishes the login. This configuration option sets the number of seconds since the most recent incoming request from that user until the user's session with the Authservice should expire. When configured to `0`, which is the default value, session expiration will not consider idle time, but can still consider timeout based on maximum absolute time since added. When both `max_absolute_session_timeout` and `max_session_idle_timeout` are zero, then sessions will never expire. These settings do not affect how quickly the OIDC tokens contained inside the user's session expire. | uint32 |
| trusted_certificate_authority | When specified, the Authservice will trust the specified Certificate Authority when performing HTTPS calls to the Token Endpoint of the OIDC Identity Provider. Optional. | string |



##### message `TokenConfig` (config/oidc/config.proto)

Defines how a token obtained through an OIDC flow is forwarded to services.

| Field | Description | Type |
| ----- | ----------- | ---- |
| header | The name of the header that authservice adds to the request when forwarding to services. The value of this header will contain the `preamble` and the token. This value is case-insensitive, as http header names are case-insensitive. Note that this value must be `Authorization` for the [Istio Authentication Policy](https://istio.io/docs/tasks/security/authn-policy/) to inspect the token. Required. | string |
| preamble | The authentication scheme of the token. For example, when the preamble is `Bearer` and `header` is `Authorization`, the following header will be added to the request to the service: `Authorization: Bearer ID_TOKEN_VALUE`. Note that this value must be `Bearer`, case-sensitive, when header is `Authorization`. Optional. | string |


