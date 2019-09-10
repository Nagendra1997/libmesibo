## Customizing MesiboUI 

You can customize every element of Mesibo chat. Everything in your application can be built on Mesibo to suit your requirements and branding. This tutorial focuses on customising your the UI in a Mesibo Android Application.You can follow a similar procedure to customise your UI in iOS,React,etc 

## PreRequisites
Before we dive into customisation,please checkout the following:
- The [Getting Started](https://mesibo.com/documentation/get-started/) guide
- Tutorial on Writing your First mesibo Enabled Application(https://mesibo.com/documentation/tutorials/first-app/)
- [Mesibo API documentation for Android](https://mesibo.com/documentation/install/android/)
- Check out our fully featured WhatsApp clone using Mesibo [here](https://mesibo.com/documentation/tutorials/open-source-whatsapp-clone/)

## The UI elements
The two basic elements that form the chat application are
- Incoming chat view
- Outgoing chat view

Mesibo chat view is a recycler view that contains the incoming chat view and outgoing chat view, which adds Items(here messages) when a message is sent or recieved.

Follow the steps below to customise your Mesibo Application UI .

## Add dependencies
Open build.gradle (app) and add the dependencies needed.

```java
dependencies {
...
implementation 'com.android.support:recyclerview-v7:25.1.0'
implementation 'com.mesibo.api:mesibo:1.0.87'

}
```
Sync Gradle and you are all set.


In order to have your own custom chat layout, you need to have one MessagingActivity class and one MessagingUiFragment class.

MessagingActivity.java loads MessagingUIFragment which has MesiboRecyclerView and Adapter. MesiboUiFragment has your custom view.
Lets create them one by one:



## 1. Create MessagingUIFragment.java

Create the MessagingUiFragment that extends MesiboMessagingFragment. 


```java
public class MessagingUiFragment extends MesiboMessagingFragment implements MesiboRecycleViewHolder.Listener {
   @Override
   public int Mesibo_onGetItemViewType(Mesibo.MessageParams messageParams, String s) {

       return 0;
   }

   @Override
   public MesiboRecycleViewHolder Mesibo_onCreateViewHolder(ViewGroup viewGroup, int i) {
       return null;
   }

   @Override
   public void Mesibo_onBindViewHolder(MesiboRecycleViewHolder mesiboRecycleViewHolder, int i, boolean b, Mesibo.MessageParams messageParams, Mesibo.MesiboMessage mesiboMessage) {

   }

   @Override
   public void Mesibo_UpdateViewHolder(MesiboRecycleViewHolder mesiboRecycleViewHolder, Mesibo.MesiboMessage mesiboMessage) {

   }

   @Override
   public void Mesibo_onViewRecycled(MesiboRecycleViewHolder mesiboRecycleViewHolder) {

   }
}
```

### Mesibo_onGetItemViewType

This method returns the type of message i.e Incoming message or Outgoing message. The returned type is then checked in `Mesibo_onCreateViewHolder()`  and then the custom view is added to recycler view accordingly.

```java
 if (messageParams.isIncoming()) {

       return MesiboRecycleViewHolder.TYPE_INCOMING;

   }
   return MesiboRecycleViewHolder.TYPE_OUTGOING;
```

##Adapter methods


### Mesibo_onCreateViewHolder
In this method type of message(Incoming and Outgoing) is checked . Based on the type you can return your own custom view or return null to load default view.

```java
  if (MesiboRecycleViewHolder.TYPE_INCOMING == type) {
           View v = LayoutInflater.from(viewGroup.getContext()).inflate(R.layout.incoming_chat_layout, viewGroup, false);

           return new IncomingMessgaeViewHolder(v);

       }
       return null;
```

### Mesibo_onBindViewHolder()

This method binds the returned view and the messages. 

```java
if (type == MesiboRecycleViewHolder.TYPE_INCOMING) {

   IncomingMessgaeViewHolder IncomingView = (IncomingMessgaeViewHolder) mesiboRecycleViewHolder;
IncomingView.mIncomingMessageTV.setText(Html.fromHtml(mesiboMessage.message));

}
```

## Create a custom row Layout
The row Layout represents the Layout of each single element displayed in the MesiboRecyclerView.
Create a file named `incoming_chat_layout.xml` and add the following to it:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
   xmlns:android="http://schemas.android.com/apk/res/android"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content">
   <FrameLayout
       android:id="@+id/incoming_layout_bubble"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:layout_marginLeft="8dp"
       android:layout_marginRight="60dp"
       android:layout_alignParentLeft="true"
       android:layout_marginTop="@dimen/_1sdp"
       android:background="@drawable/balloon_incoming_normal">


      
       <TextView
           android:id="@+id/textview_message"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:paddingLeft="@dimen/_15sdp"
           android:paddingRight="@dimen/_15sdp"
           android:paddingTop="@dimen/_1sdp"
           android:paddingBottom="@dimen/_5sdp"
           android:textSize="@dimen/_15sdp"
           android:textColor="@color/black_overlay"
           android:autoLink="web"
           android:text="Text Message goes here"
           android:visibility="visible"/>
       </LinearLayout>
   </FrameLayout>

</RelativeLayout>


}
```
Similarly for outgoing message another custom row layout can be created.

### Creating the ViewHolder
The MesiboRecyclerView uses a ViewHolder to store the references to the relevant views for one entry in the RecyclerView. This solution avoids all the findViewById() method calls 

```java
public class IncomingMessgaeViewHolder extends MesiboRecycleViewHolder {
   TextView mIncomingMessageTV
   View mViewIncomingMessage;

   public IncomingMessgaeViewHolder(View v) {
       super(v);
       mIncomingMessageTV = v.findViewById(R.id.textview_message);
            mViewIncomingMessage = v;
   }
}
```

### Full MessagingUIFragment() code
```java
public class MessagingUiFragment extends MesiboMessagingFragment implements MesiboRecycleViewHolder.Listener {

public class IncomingMessgaeViewHolder extends MesiboRecycleViewHolder {

   TextView mIncomingMessageTV
   View mViewIncomingMessage;

   public IncomingMessgaeViewHolder(View v) {
       super(v);
       mIncomingMessageTV = v.findViewById(R.id.textview_message);
            mViewIncomingMessage = v;
   }
}




   @Override
   public int Mesibo_onGetItemViewType(Mesibo.MessageParams messageParams, String s) {

      if (messageParams.isIncoming()) {

       return MesiboRecycleViewHolder.TYPE_INCOMING;

   }
   return MesiboRecycleViewHolder.TYPE_OUTGOING;


   }

   @Override
   public MesiboRecycleViewHolder Mesibo_onCreateViewHolder(ViewGroup viewGroup, int i) {
      if (MesiboRecycleViewHolder.TYPE_INCOMING == type) {
           View v = LayoutInflater.from(viewGroup.getContext()).inflate(R.layout.incoming_chat_layout, viewGroup, false);

           return new IncomingMessgaeViewHolder(v);

       }
       return null;

   }

   @Override
   public void Mesibo_onBindViewHolder(MesiboRecycleViewHolder mesiboRecycleViewHolder, int i, boolean b, Mesibo.MessageParams messageParams, Mesibo.MesiboMessage mesiboMessage) {
if (type == MesiboRecycleViewHolder.TYPE_INCOMING) {

   IncomingMessgaeViewHolder IncomingView = (IncomingMessgaeViewHolder) mesiboRecycleViewHolder;
IncomingView.mIncomingMessageTV.setText(Html.fromHtml(mesiboMessage.message));

}

   }

   @Override
   public void Mesibo_oUpdateViewHolder(MesiboRecycleViewHolder mesiboRecycleViewHolder, Mesibo.MesiboMessage mesiboMessage) {

   }

   @Override
   public void Mesibo_onViewRecycled(MesiboRecycleViewHolder mesiboRecycleViewHolder) {

   }
}
```

Create `activity_messaging.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:app="http://schemas.android.com/apk/res-auto"
   xmlns:tools="http://schemas.android.com/tools"
   android:id="@+id/chat_root_layout"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   android:fitsSystemWindows="true"
   android:theme="@style/MesiboAppTheme"
   >

   <android.support.design.widget.AppBarLayout
       android:layout_width="match_parent"
       android:layout_height="wrap_content"
       android:theme="@style/AppTheme.AppBarOverlay">    
   </android.support.design.widget.AppBarLayout>

   <!-- https://stackoverflow.com/questions/32956071/add-views-below-toolbar-in-coordinatorlayout
    layout_behavior is important
   -->
   <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
       android:id="@+id/fragment_container"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       app:layout_behavior="@string/appbar_scrolling_view_behavior" />

   <!-- <include layout="@layout/chat_box_new" /> -->
</android.support.design.widget.CoordinatorLayout>
```
### Create MessagingActivity.java

Create a messaging activity and get the peer(destination) through intent. Then you need to load a 

**1.Create one YourMessagingActivity.class**

This activity is called from where you want to launch your own customized message ui. 

Pass value of peer(where the message is to be sent) to this activity in intent and get ‘peer’ here in this activity.

```java
Bundle args = getIntent().getExtras();
String peer = args.getString("peer"); 



public class MessagingActivity extends AppCompatActivity implements MesiboMessagingFragment.FragmentListener {


   MessagingUIFragment mFragment = null;
   private Mesibo.MessageParams mParameter=null;
private Mesibo.UserProfile mUser=null;


   @Override
   protected void onCreate(Bundle savedInstanceState) {
       //getWindow().requestFeature(Window.FEATURE_ACTION_MODE_OVERLAY);

       super.onCreate(savedInstanceState);
       Bundle args = getIntent().getExtras();
       if(null == args) {
           return;
       }

       //TBD, this must be fixed
       if(!Mesibo.isReady()) {
           finish();
           return;
       }

 
       String peer = args.getString("peer");
    
       mUser = Mesibo.getUserProfile(peer);

       if(null == mUser) {
           mUser = new Mesibo.UserProfile();
           mUser.address = peer;
           mUser.name = peer;
           Mesibo.setUserProfile(mUser, false);
       }

       mParameter = new Mesibo.MessageParams(peer, groupId, Mesibo.FLAG_DEFAULT, 0);

       setContentView(R.layout.activity_messaging);
       startFragment(savedInstanceState);

   }

   private void startFragment(Bundle savedInstanceState) {
       // However, if we're being restored from a previous state,
       // then we don't need to do anything and should return or else
       // we could end up with overlapping fragments.
       if (findViewById(R.id.fragment_container) == null || savedInstanceState != null) {
           return;
       }

       // Create a new Fragment to be placed in the activity layout
       mFragment = new MessagingUIFragment();

       // In case this activity was started with special instructions from an
       // Intent, pass the Intent's extras to the fragment as arguments
       mFragment.setArguments(getIntent().getExtras());

       // Add the fragment to the 'fragment_container' FrameLayout
       getSupportFragmentManager().beginTransaction()
               .add(R.id.fragment_container, mFragment).commit();
   }
}


```
### Conclusion

