# Zigate enhancements requests

The purpose is to document to describe missing key points for Zigate futur.

Globally and except if Zigate FW has to do the job, no other feature is required. Any missing support should be handled by the plugin thru 8002/0530 key commands that allow to really talk "zigbee" without additional layer.

1. Backup & restore current network "mapping"

   - To move from one zigate to another.
   - To restore to last known clean state. Repairing all devices is not an option. Not always easily possible and makes zigate experience just bad.

1. Being able to remove a device from zigate network WITHOUT the device

   - The device might be dead and therefore will always use an internal slot.
   - The device may already have been moved to another network.

1. Need to known current network filling status

   The point there for end user is to understand if zigate limit is reached
   instead of wasting time trying to include and reinclude.

1. Regenerate PDM

   From an external list of devices, regenerate PDM from scratch to deal
   with any internal corruption and avoid full repairing.

1. Not key but useful: Zigate to notify plugin when network is closed

   Since this can be triggered by external device (ex: Profalux case), the
   plugin should be informed that process as stopped.

   Note: Current work around is to often poll status while in pairing mode.
