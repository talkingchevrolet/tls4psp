This code was tested in an emulator. May suffer on real hardware.
tls4psp is a STATIC LIBRARY that allows for TLS connections and a variety of requests.
It is made for compilation. You use the .h and compile just one file, the .a. But it requires specific psp libs to be linked IN YOUR EBOOT.PBP, SEPERATELY OF THE LIBRARY!
Namely:
    pspnet
    pspnet_inet
    pspnet_resolver
    pspuser
    psppower
    pspnet_apctl
    plus lc and lgcc as non PSPSDK libs.
Now, other than that, it is designed to be relatively low friction.
Your 4 commands:
int tls_init(void);

int tls_connect(const char *host,
                uint16_t port,
                const char *request,
                const char *body,
                unsigned char *response,
                size_t resp_size,
                int showdebug,
                int preset);

int tls_destroy(void);

IF USING TLS4PSP ABOVE 1.0.5

void tls_set_lightweight(int enable)

You call tls_init, tls_connect, and tls_destroy, in that order. This has been smoothed out. When using tls4psp above 1.0.5, you can simply call tls_init(), and then tls_connect() as many times as you want, before going to tls_destroy().
This uses very hard ciphers and curves for the Allegrex. This means there is a lot of compatibility, but handshakes can take up to 20 seconds on older versions!
Testing shows roughly 10 - 15 seconds, but you never know.
Luckily, for more efficiency, on versions above 1.0.5, you can call tls_set_lightweight(1) to use lighter ChaChaPoly ciphers, and sequential tls_connect calls as well.
ChaChaPoly increases CPU efficiency, and the ability to run sequential tls_connect() calls without tls_destroy() directly cuts connection times.
On versions above 1.0.5, handshake time was reduced from 10 - 15 seconds to 3 - 5 seconds.

EXAMPLES:
    int bytes1 = tls_connect(
        "www.letsencrypt.org",
        443,
        "/",
        NULL,
        response1,
        sizeof(response1) - 1,
        1,
        1
    );
    NOTE: body can be set to NULL because preset 1 is used (GET). Body is only needed for POST.
    "/" just means there is nothing else to the url. When doing google.com/humans.txt, for example, /humans.txt would be where / is.
    to show my point further:
        int bytes2 = tls_connect(
        "www.postman-echo.com",
        443,
        "/gzip",
        NULL,
        response2,
        sizeof(response2) - 1,
        1,
        1
    );
    NOTE: gzip support is only for tls4psp v1.2.7
