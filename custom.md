## Customizing MesiboUI 
You can customize almost every element of Mesibo chat.
Basic elements are
- Incoming chat view
- Outgoing chat view

Mesibo chat view is simply a recycler view which adds Items(here messages) when ever a message is received or sent.

Follow these steps:

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

## Create MessagingUIFragment.java

Create the MessagingUiFragment that extends MesiboMessagingFragment. Here you have a recyclerView that is populated whenever there is a message sent or received.
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
   public void Mesibo_oUpdateViewHolder(MesiboRecycleViewHolder mesiboRecycleViewHolder, Mesibo.MesiboMessage mesiboMessage) {

   }

   @Override
   public void Mesibo_onViewRecycled(MesiboRecycleViewHolder mesiboRecycleViewHolder) {

   }
}
```

### Mesibo_onGetItemViewType

This method returns the type of message i.e Incoming message or Outgoing message. The returned type is then checked in Mesibo_onCreateViewHolder()  and then the custom view is added to recycler view accordingly 

```java
 if (messageParams.isIncoming()) {

       return MesiboRecycleViewHolder.TYPE_INCOMING;

   }
   return MesiboRecycleViewHolder.TYPE_OUTGOING;
```
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


