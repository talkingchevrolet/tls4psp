As of Jan 2026, this code is completely untested.
tls4psp allows you to make HTTP calls through TLS 1.2 ECDHE. It's pretty much the bare minimum, but infinitely better than the current stack.
BUT, there is always a catch.
This cannot change existing Sony functions. This will not revive the browser on its own.
So what does it do?
It allows YOU to do that.
You call tls_connect() to make HTTP requests inside your program. It works in one shot. You cannot run two at once.
anatomy of tls_connect():
host - string of the server IP or hostname
port - port number (usually 443 for HTTPS, but you can be lazy and pass 0 to automatically do 443. Do not pass NULL. Please pass a valid port.)
request - HTTP request string to send (can be any type of request, try stress testing it)
response - buffer where the function writes the server response
resp_size - size of the response buffer
showdebug - set to 1 to see some information. Or you can just set it to 0.
tls_connect(const char *host, uint16_t port, const char *request,
                unsigned char *response, size_t resp_size, int showdebug);

As with any code, you can make errors, or maybe the network isn't on your side today. The function will let you know of these.
ERR LIST:
-1: Network socket was not created.
-2: Could not connect to server.
-3: TLS handshake failed.
-4: Certificate could not be verified.
-5: Failed to create resolver for hostname.
-6: Resolver was made, but could not resolve hostname.
-7: Failed to send GET request.
-8: Failed to initialize SSL.
-9: Overflow risk (buffer passed by the user is too small)
-10: Certificate could not be parsed (different from verification).
-11: DRBG could not initiate.
-12: TLS handshake timed out. This doesn't actually mean much sinister, but check your internet connection.
Errors -11 and -12 occur during module_start() and may prevent module initialization.
