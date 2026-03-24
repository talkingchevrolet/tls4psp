tls4psp is a small static library that allows for basic TLS and HTTP support on the PSP. It has been tested in an emulator, but may suffer on real hardware, especially on the PSP 1000 model, because of its low RAM.
tls4psp is designed to be simple, and simple it is. It abstracts a large set of functions into a single command.
On 1.0.5 and above, your flow will look like this:
tls_init()
tls_connect()
tls_connect()
tls_destroy() //when you are finished
Note that use of 1.0.0 is not advised, as it has a small scope of certificates and no gzip support.
tls_connect() CANNOT BE CALLED CONCURRENTLY, but it can be called repeatedly within the same thread.
You do NOT have to call tls_destroy() after every tls_connect() call.
Of course, there are also optional commands.
When using tls4psp 1.2.5 and above, you have the option to do
tls4psp_set_lightweight(int enable).
If you set this to 1, you trade AES ciphers for lighter ChaChaPoly ciphers.
if you don't set it, you get the default set.
When using tls4psp 1.3.8 and above, you get another command.
int tls_poll(const char *host,
                uint16_t port,
                const char *request,
                const char *body,
                unsigned char *response,
                size_t resp_size,
                int showdebug,
                int preset,
                int amount,
                int timing);
This allows for polling obviously, but it is blocking. To use it, fill out the first few commands exactly like in tls_connect, then
set amount to however many times you want to poll and timing (in seconds) on how long to wait between polling.
So, to use tls_connect:
first set a response buffer. this is where your data will be written.
unsigned char response[2048]; //this leaves some space in RAM to write your response.
tls_init() // you MUST have this before tls_connect()
int tls_connect(const char *host, //you set this to the host, for example www.example.com
                uint16_t port, //you set this as the port used. Set this to 443, or set it to 0 for a shortcut.
                const char *request, //set this as the path. For example, if you are polling example.com/gzip, you would put /gzip in the request section.
                const char *body, //This is only for POST requests. For HTTP GET, leave this as NULL.
                unsigned char *response, // the response buffer to be used
                size_t resp_size, //the size of your buffer
                int showdebug, //set this to 1 or 0, this is a useless artifact.
                int preset); // set to 1 to use HTTP GET, set to 2 to use HTTP OPTIONS, set to 3 to use HTTP POST.
When reading, do not directly read from tls_connect(), as that only shows the bytes as a number, not the bytes themselves.
Read from the response buffer you set, as that is where the data is written. Make sure to null-terminate your response by doing
response[bytes] = '\0';, with bytes being whatever you named your call of tls_connect().
If the response is only "H", this is caused by a response buffer that is too small. Increase it.
Now, you are all set, but what if you encounter errors?
//Network errors
#define NET_ERR_SOCKET_FAILED              -1 //this means the socket has enough memory but failed to create. Shouldn't happen.
#define NET_ERR_SOCKET_CONNECTION_FAILED   -2 //Failed to connect to the resolved host
#define NET_ERR_RESOLVE_GONE               -3 //Failed to create resolver instance
#define NET_ERR_FAILED_TO_RESOLVE          -4 //Host could not be resolved. Did you set the host properly?
#define NET_ERR_SOCKET_NO_MEM              -5 //Not enough free RAM to make a socket. Should never happen.
#define NET_ERR_APCTL_TIMEOUT              -6 //Timed out when connecting to the internet.
//tls errors
#define TLS_ERR_STARTUP_ERROR              -50 //Misc error
#define TLS_ERR_CERT_VERIFY_FAILED         -51 // note: -9984 is a related error from mbedtls. occurs when the website doesn't show a cert we recognize.
#define TLS_ERR_FAILED_TO_SEND             -52 //For whatever reason, mbedtls could not send from the socket
#define TLS_ERR_CANT_INITIALIZE            -53 //For whatever reason, the server was connected to, but TLS cannot initialize
#define TLS_ERR_RESPONSE_WOULD_OVERFLOW    -54 //response buffer is malformed or too large/small
#define TLS_ERR_CERT_FAILED_PARSE          -55 //the hardcoded certificates failed to be parsed and read.
#define TLS_ERR_DRBG_FAILED_SEEDING        -56 //the DRBG could not seed from the entropy source. should not happen.
#define TLS_ERR_HANDSHAKE_TIMEOUT          -57 //Handshake timed out. Check your internet speed or CPU.
#define TLS_ERR_INVALID_PRESET             -58 //Invalid value put in the preset section of tls_connect().
#define TLS_ERR_REQUEST_NO_MEM             -59 //Not enough free RAM to store and send the request. Should never happen.
#define TLS_ERR_CURVE_FAILED               -60 //TLS curves could not be initialized. shouldn't happen.
#define TLS_ERR_INVALID_CIPHER_SELECTION   -61 //invalid value passed to tls_set_lightweight(). It only accepts 0 or 1.
#define TLS_ERR_BAD_ENTROPY_SELECTION      -62 //Not caused by the user, but for whatever reason, DRBG cannot accept the entropy source.
//gzip errors
#define PARSE_ERR_GZIP_DECOMPRESSOR_NO_MEM -100 //No free RAM to give the gzip decompressor. Should never happen.
#define PARSE_ERR_GZIP_PRODUCT_NO_MEM      -101 //Not enough free RAM to store the product of decompression. Should never happen.

Note: This is NOT thread-safe, and it is blocking. For doing multi-threaded operations, it is best to have a workaround, like mutexes, global buffers, it is up to you.
Connections are closed after each request (no keep-alive support).
