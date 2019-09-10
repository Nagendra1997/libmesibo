## Tutorial on using  the Mesibo C/C++ API

The Mesibo C/C++ library can be seamlessly integrated into your application on any platform, with custom extensions/wrappers. To compile your C/C++ application with Mesibo, you need to link it with the Mesibo  Library File. On any Linux based system, Mesibo is available as a shared library(.so) which allows you to use it from any application OR languages of your choice like C, C++, Python, PHP, Matlab, etc.

In this part, we will create a simple real-time app using C/C++.

### OS requirements
- CentOS / RedHat 7.x or above
- Debian / Ubuntu
- Mac OS
- Raspberry Pi


### Prerequisites

You MUST go through the following prerequisites before you read further.

- Installed the Mesibo Real-Time C/C++ shared Library by referring to the installation instructions 
  [here](https://mesibo.com/documentation/install/linux/) 

- Read the [Preparation Guide](https://mesibo.com/documentation/tutorials/first-app/)

- Read the [Anatomy of a Mesibo Application](https://mesibo.com) about using the Mesibo API and listener class

- Basic knowledge of writing and compiling C/C++ code


### First C/C++ App

Now let’s quickly start coding:

1. Open a Code Editor
2. Include <mesibo.h> in your file
3. Add mesibo initialization code in your init function.

```C++
#include <mesibo.h>

```

```C++

void mesibo_init(){
  CNotify *n = new CNotify();
  IMesibo *m_api = query_mesibo("/tmp");
  n->set_api(m_api);

  m_api->set_notify(0, n, 1);
  
  // Put your AUTH_TOKEN obtained from Mesibo Console
  m_api->set_credentials("aea59d3713701704bed9fd5952d9419ba8c4209a335e664ef2g");

  if (0 != m_api->set_database("mesibo.db")) {
    fprintf(stderr, "Database failed\n");
    return -1;
  }
  
  // Put your APP_ID obtained from Mesibo Console
  m_api->set_device(1, "MyDeviceId", myappid, "1.0.0");
  m_api->start();
 
  return 0;
  }

```

As explained in [Anatomy of Mesibo Application](https://mesibo.com), Mesibo invokes a class of Listeners for various events. 

Derive from the `INotify` class to implement listeners as shown below.

```C++

class CNotify : public INotify {
  IMesibo *m_api;

  public:
  void set_api(IMesibo *api) { m_api = api; }


  // You will receive the connection status here
  int on_status(int status, uint32_t substatus, uint8_t channel,
                const char *from) {
    ERRORLOG("===> on_status: %u %u\n", status, substatus);
    return 0;
  }


  // Invoked on receiving a new message or reading database messages
  // You will receive messages here.
  int on_message(tMessageParams * p, const char *from, const char *data,
                 uint32_t len) {
    int printlen = len;
    if (printlen > 64) printlen = 64;

    ERRORLOG(
        "===> test app message received: uid %u status %d channel %d type %u "
        "id %" PRIx64 " refid %lu groupid %u, when %" PRIu64
        " from %s, flag: %x len %d: %.*s\n",
        p->uid, p->status, p->channel, p->type, p->id, p->refid, p->groupid,
        p->when, from, p->flag, len, printlen, data);

    return 0;

  }

  // Invoked when the status of the outgoing or sent message is changed
  // You will receive the status of sent messages here
  int on_messagestatus(tMessageParams * p, const char *from, int last) {
   ERRORLOG(
        "===> on_messagestatus status %u id %u when %u ms (%u %u) from: %s\n",
        p->status, p->id, m_api->timestamp() - p->when, m_api->timestamp(),
        p->when, from ? from : "");
    return 0;

  }

  int on_messagebundle(tMessageParams *p, const char *from, tMessageBundle *m) {
    return 0;
  }

  int on_error(uint8_t error) { return 0; }

  int on_activity(tMessageParams *p, const char *from, uint32_t event) {
    return 0;
  }

  int on_location(tMessageParams *p, const char *from, float lat, float lon,
                  const char *title, const char *message, const char *thumbnail,
                  int tnlen) {
    return 0;
  }

  int on_file(tMessageParams *p, const char *from, uint32_t fileflag,
              int filesize, const char *url, const char *thumbnail, int tnlen,
              const char *title, const char *message, const char *launchurl,
              const char *localpath) {
    return 0;
  }

  int on_call(uint32_t peerid, uint32_t callid, int status, const char *data,
              int datalen, uint64_t flags) {
    return 0;
  }

  int on_key(const char *key, const char *value) {
    return 0;
  }

  int on_contact(const tContact *c) {
    return 0;
  }

  int on_rtc(int type, uint32_t flags, uint32_t peerid, const char *sdp,
             int len) {
    return 0;
  }

  int on_server(int type, const char *server, const char *username,
                const char *password) {
    return 0;
  }

};


```
That’s it - you are now ready to receive your first real-time message.

## Testing your C/C++ application

1. Compile and Run your code 

It is recommended that you use a modern C/C++ compiler such as gcc(GCC 4.x or above) or clang. You can compile your code by linking the Mesibo Shared Library as shown below:

```bash
g++ my_first_app.cpp -o myfirstapp -lmesibo64

```
Run the output file
```bash
./myfirstapp
```

2. `on_status` should cycle through various status information. Finally, you should receive status=1 which indicates that your app is successfully connected to the mesibo real-time server and ready to send and receive real-time messages.

3. Since we do not have any other users right now, we will use **mesibo console** to send a test message. In a later section, we will learn how to send messages from the code itself.

- Go to **Console ->Application->Users**. You should see the user you have created.

- Go to user details by clicking the `Edit` button. Scroll down, you will see a section to `Message User`
Enter 1000 (or anything) in `From` field, check `Create This User` checkbox, type message and click on `Send`.

4. You will instantly receive this message in your console/terminal in `on_message` listener.

## Sending Messages
In the previous section, we have used mesibo console to send a message. Now we will quickly learn how to send messages from the code itself. To send messages, we will use `message` real-time API for which we will need destination user, message-id and the message itself.

Invoke the following function anywhere from your code to send a text message. 

```C++
int send_text_message(IMesibo* m_api,const char* to,const char * message){

      tMessageParams p = {};
      p.id = m_api->random32();

      m_api->message(&p, to, message, strlen(message));
}

```
For example, Call this function from `on_status` to send a message when you are online.

```C++
  int on_status(int status, uint32_t substatus, uint8_t channel,
                const char *from) {
    fprintf(stderr,"===> on_status: Mesibo is Online! Sending Message to TestUser .. ");
    if (status == 1) {
        send_text_message(m_api, "TestUser", "Hello from Mesibo C/C++. I am Online!");
    }
    return 0;

```

That’s it! Try it out by creating two users and send messages to each other by using the above function.
