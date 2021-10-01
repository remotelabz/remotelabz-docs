# The Messenger manager

The Front and the Worker exchange messages from bundle crestic-urca/remotelabz-message-bundle. The Front send Action message and the Worker answer with Stage message.
The action is related instance and from version 2.4.0 :

 - Create (Lab)
 - Delete (Lab and Device)
 - Start (Device)
 - Stop (Device)
 - Export (Device).

The worker answers, respectively, with the state : 

 - Created
 - Deleted
 - Started
 - Stopped
 - Exported.

Another state is added : the state Error.
