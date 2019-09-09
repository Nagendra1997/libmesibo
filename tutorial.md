## Tutorial on using  the Mesibo C/C++ API

Mesibo on Linux is available as a shared library(.so) which allows you to use it from any application OR languages of your choice like C, C++, Python, PHP, Matlab etc.

### OS requirements
- CentOS / RedHat 7.x or above
- Debian / Ubuntu
- Mac OS

This is a simple tutorial for sending/recieving a text-message using the entirely open-source and Real-Time Mesibo C/C++ Library. 

### Prerequisites
Before you begin,
- Please ensure that you have installed the Mesibo Real-Time C/C++ shared Library by referring to the installation instructions [here](https://mesibo.com/documentation/install/linux/)
- Also,please go through [Get Started](https://mesibo.com/documentation/get-started/) guide to gain an understanding about Mesibo
- Refer to [Write your First mesibo Enabled Application](https://mesibo.com/documentation/tutorials/first-app/) for information on how to create users and obtain access token

Let's get Real-Time !

**1. Create your application**

- Create a new application from the [Mesibo console](https://mesibo.com/console)

- Create Users (Endpoints)

Create users from the console by clicking on ‘New User’ button from the Application settings page. Create two users  named `TestUser1` and `TestUser2` and generate  access tokens for the users with your App Id.For example,the App Id could be `my_cpp_app` .


Please refer to our tutorial [Write your First mesibo Enabled Application](https://mesibo.com/documentation/tutorials/first-app/) about creating users for more information.

We will be communicating between `TestUser1` and `TestUser2`.
Note the Auth token and the App ID for users `TestUser1` and `TestUser2`.


**2. Include the Mesibo header file**

The required header file to use Mesibo in your C/C++ project is `mesibo.h`. Ensure that this file is present in your path while compilation.
You can include the Mesibo header file like so in your file:

```C++
#include <mesibo.h>
```

Mesibo invokes various Listeners for various events.
For example, when you receive a message, receive an incoming call,etc.
CNotify is a class of listeners that can be invoked to get real-time notification of events . INotify is the base class definition from which CNotify is derived from. You can override the notify class behaviour to perform as per your requirements.

```C++
class CNotify : public INotify {
  IMesibo *m_api;

  public:
  void set_api(IMesibo *api) { m_api = api; }


  // Invoked on receiving a new message or reading database messages
  // You will receive messages here.
  int on_message(tMessageParams * p, const char *from, const char *data,
                 uint32_t len) {
    int printlen = len;
    if (printlen > 64) printlen = 64;

   fprintf(stderr,"===> on_message: You have recieved a message %.*s\n :printlen, data);

    return 1;
  }

  // Invoked when the status of outgoing or sent message is changed
  // You will receive status of sent messages here
  
  int on_messagestatus(tMessageParams * p, const char *from, int last) {
    if(p->status == MESIBO_MSGSTATUS_SENT)
        fprintf(stderr,"===> on_messagestatus :"Your message has been sent!");
    return 0;
  }

  // You will receive the connection status here
  int on_status(int status, uint32_t substatus, uint8_t channel,
                const char *from) {
    if(status ==  MESIBO_STATUS_ONLINE)  {              
    fprintf(stderr,"===> on_status: Mesibo is Online!  ");
    }
    return 0;
  }

}

```


You need to initialise Mesibo for `TestUser1`. Enter the `AUTH TOKEN` and `APP ID`
for `TestUser1` as noted in Step-2

```C++
//For TestUser1
#define AUTH_TOKEN "aea59d3713701704bed9fd5952d9419ba8c4209a335e664ef2e"
#define APP_ID "my_cpp_app"
```
You need to pass your `AUTH_TOKEN` to the function set_credentials and your `APP_ID` to the function set_device,during initialisation.

Initialization code :
```C++
int main(){
  CNotify *n = new CNotify();
  IMesibo *m_api = query_mesibo("/tmp");
  n->set_api(m_api);

  m_api->set_notify(0, n, 1);
  m_api->set_credentials(AUTH_TOKEN);

  if (0 != m_api->set_database("mesibo.db")) {
    fprintf(stderr, "Database failed\n");
    return -1;
  }

  m_api->set_device(1, "MyDeviceId", APP_ID, "1.0.0");

  m_api->start();

  return 0;
  }


```
You can run the same code to initialise `TestUser2`,you just need to change the `AUTH_TOKEN` and `APP_ID` that is linked with the user `TestUser2` . 

```C++
//For TestUser2
#define AUTH_TOKEN "2059cdd3e60de6482b0065fc12eb03d601cce7a8c60396e8fe446db9c"
#define APP_ID "my_cpp_app"
```

**3. Sending Messages**

We will be sending a message from `TestUser1` to `TestUser2`.

To send messages,you can use message real-time API for which you will need destination user, message id which should be included in the structure defined by `tMessageParms` and the message itself.

Invoke the following function from your code to send a text message
```C++
int send_text_message(IMesibo* m_api,const char* to,const char * message){

      tMessageParams p = {};
      p.id = m_api->random32();

      m_api->message(&p, to, message, strlen(message));
}



```
For example,Call this function from on_status to send a message when you are online.
```C++
  int on_status(int status, uint32_t substatus, uint8_t channel,
                const char *from) {
    fprintf(stderr,"===> on_status: Mesibo is Online! Sending Message to TestUser2 .. ");
    if (status == 1) {
        send_text_message(m_api, "TestUser2", "Hello from Mesibo C/C++");
    }
    return 0;

```

Summing up,The complete program should look like below
```C++
// Test file for using Mesibo C/C++ library
// TestUser1

#include <inttypes.h>
#include <mesibo.h>
#include <stdlib.h>
#include <string.h>
#include <termios.h>
#include <unistd.h>

int gDebugEnabled = 1;

int send_text_message(IMesibo *m_api, const char *to, const char *message) {
  tMessageParams p = {};
  p.id = m_api->random32();

  m_api->message(&p, to, message, strlen(message));
}


int keypress() {
  struct termios old_state, new_state;
  int c;

  tcgetattr(STDIN_FILENO, &old_state);
  new_state = old_state;

  new_state.c_lflag &= ~(ECHO | ICANON);
  new_state.c_cc[VMIN] = 1;

  tcsetattr(STDIN_FILENO, TCSANOW, &new_state);

  c = getchar();

  /* restore the saved state */
  tcsetattr(STDIN_FILENO, TCSANOW, &old_state);
  return c;
}

class CNotify : public INotify {
 IMesibo *m_api;

 public:
  void set_api(IMesibo *api) { m_api = api; }

  int on_message(tMessageParams *p, const char *from, const char *data,
                 uint32_t len) {
    int printlen = len;
    if (printlen > 64) printlen = 64;

    fprintf(
        stderr,
        "===> test app message received: uid %u status %d channel %d type %u "
        "id %" PRIx64 " refid %lu groupid %u, when %" PRIu64
        " from %s, flag: %x len %d: %.*s\n",
        p->uid, p->status, p->channel, p->type, p->id, p->refid, p->groupid,
        p->when, from, p->flag, len, printlen, data);

    return 1;
  }

  int on_messagestatus(tMessageParams *p, const char *from, int last) {
    fprintf(
        stderr,
        "===> on_messagestatus status %u id %u when %u ms (%u %u) from: %s\n",
        p->status, p->id, m_api->timestamp() - p->when, m_api->timestamp(),
        p->when, from ? from : "");
    return 0;
  }

  int on_status(int status, uint32_t substatus, uint8_t channel,
                const char *from) {
    fprintf(stderr, "===> on_status: %u %u\n", status, substatus);
    if (status == 1) send_text_message(m_api, "TestUser2", "Hello from Mesibo C/C++");
    return 0;
  }
};

//Replace these values for a different user
#define AUTH_TOKEN "3927c2ef921d619a920068000c168df7bebcc3a6f59f7928feaa4ef1d"
#define APP_ID "dialogflowmesibo"

int main() {
  CNotify *n = new CNotify();
  IMesibo *m_api = query_mesibo("/tmp");

  n->set_api(m_api);

  m_api->set_notify(0, n, 1);
  m_api->set_credentials(AUTH_TOKEN);

  if (0 != m_api->set_database("mesibo.db")) {
    fprintf(stderr, "Database failed\n");
    return -1;
  }

  m_api->set_device(1, "MyDeviceId", APP_ID, "1.0.0");
  m_api->start();

  keypress();
  return 0;
}

```
Note, the function keypress() is called to exit the program when you press the `Enter` button ,until then the Mesibo application thread will be active. 



### Compilation
It is recommended that you use a modern C/C++ compilers such as gcc(GCC 4.x or above) or clang .You can compile your code like below by including the library path to the Mesibo Shared Library  :

```bash
g++ testuser1.cpp -o user1 -lmesibo64
g++ testuser2.cpp -o user2 -lmesibo64
```
Run the files
```bash
./user2
```
Note that the Connection status for `TestUser2` is online . You can verify it from the console of your app.
Now ,you can send a message from `TestUser1`. Run
```
./user1
```

You should recieve the message "Hello from Mesibo C/C++" at `TestUser2` and the same should be printed out in your terminal.
You can also send messages from the Mesibo console.   
