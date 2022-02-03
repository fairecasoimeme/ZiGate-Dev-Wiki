# Zigate enhancements requests

The purpose is to document to describe missing key points for Zigate futur.

Globally and except if Zigate FW has to do the job, no other feature is required. Any missing support should be handled by the plugin thru 8002/0530 key commands that allow to really talk "zigbee" without additional layer.

1. Backup & restore current network "mapping"

   - To move from one zigate to another.
   - To restore to last known clean state. Repairing all devices is not an option. Not always easily possible and makes zigate experience just bad.

  [PP] Advise to check here: https://github.com/zigpy/open-coordinator-backup this is currently implemented in some ZNP (Texas) and bellows (EZSP) keys, where basically they get access to the entire NVRAM. At then end you can backup on one brand and restore on an other brand in a full transparency.

1. Being able to remove a device from zigate network WITHOUT the device

   - The device might be dead and therefore will always use an internal slot.
   - The device may already have been moved to another network.

   [PP] https://community.nxp.com/t5/Wireless-Connectivity/JN5169-issue-on-gateway-coordinator-to-remove-its-endevice/m-p/652970

1. Need to known current network filling status

   The point there for end user is to understand if zigate limit is reached
   instead of wasting time trying to include and reinclude.

1. Regenerate PDM

   From an external list of devices, regenerate PDM from scratch to deal
   with any internal corruption and avoid full repairing.
   
   [PP] shall it be simply the Restore which then re-create the PDM ?

1. Not key but useful: Zigate to notify plugin when network is closed

   Since this can be triggered by external device (ex: Profalux case), the
   plugin should be informed that process as stopped.

   Note: Current work around is to often poll status while in pairing mode.
   
   [PP] This is not part of the norm, and I wonder if you have such capability. For instance you can Open the network on one particular device. In that case, the Coordinator is not aware and there is not Zigbee norm (Cluster) to query the openess of the network.
