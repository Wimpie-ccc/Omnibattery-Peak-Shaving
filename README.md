# Omnibattery-Peak-Sheaving
Fist of all a legal message: Use at own risk, no warranty what so ever. Not even warranty that this works!

This flow is a rewrite of 2 blueprints for Omnibattery: "Peak shaving limit sync" and "Peak shaving recharge to SOC".
It has the following features:
- automatically adjust "peak limit" (omnibattery_capacity_protection_limit) with the new value if you went 
  over the old one.
- possibility to use an offset relative to the current P1 monthly peak. eg: your P1 meter rapports a peak of 3000W, 
  with an offset of 200, Omnibattery will try to limit the peak to 2800W.
- possibility to set a minimum peak. eg: 2500W for people living in Flanders.
- possibility to define a range where the battery will be recharged while peakshaving is active (SOC Floor & SOC Target).
- possibility to use an offset relative to the current P1 monthly peak for recharging the battery while peak shaving 
  is active. eg: your current P1 monthly peak is 3000W, with an offset of 300W, Omnibattery will try to recharge 
  the battery while never exceding a grid power draw of 2700W.
- when peak shaving recharging ends (SOC > SOC Target), it will set the idle grid power setpoint (omnibattery_pd_target_grid_power) to a user 
  adjustable value.

Requirments (beside the things you need to run Omnibattery):
- Node-RED
- a basic understanding how node-RED works
- peak shaving is enabled in Omnibattery (Control page), and a sane value for "SOC Threshold". I use 32%.
  
Install:
Import the flow in de usual way.

Usage:
After import and deploy, nothing will happen. Both HA action nodes are not connected. You will have to connect them 
so they can recieve messages and control Omnibattery.

Settings can be found in the settings group. Some are the same as those found in the 2 blueprints I used as inspiration.
- Monthly Peak Offset: Contains an offset value, relative to the P1 meter current month peak. This offset ensures that 
                       the P1 month peak won't creep up. eg P1 peak is 3000W, offset is 200W, the value in 
                       "omnibattery_capacity_protection_limit" will be 2800W. Default : 200W
- Min Monthly Peak: Some (all ?) regions have a minimum peak value that the grid provider will always bill. eg: 2500W
                    for Fluvius (Flanders). Default : 2500W 
- SOC Floor: Charge of the battery (in percent) when Omnibattery will start recharging the battery. Value could be 
             from 13% to ...(< 100%), but MUST be smaller than "SOC Target". Peak shaving needs to be active for this 
             to work. Default : 29%
- SOC Target: Charge of the battery (in percent) when Omnibattery will stop recharging the battery. Value could be 
             from 14% to ...(100%), but MUST be bigger than "SOC Floor". Default : 30%
- Charge Grid Offset: Contains an offset value, relative to the P1 meter current month peak. This offset is used to 
                      calculate the setpoint by which Omnibattery will recharge the battery while peakshaving is active.
                      This offset value MUST be bigger than the "Monthly Peak Offset". Default : 300W
- Idle Grid Target: Value at which Omnibattery will try to regulate the grid power draw when peak shaving is not active.
                    Default : 0W


How does this work in practice with the default values used above?

Assume P1 peak is 3000W, home uses 500W, battery Charge is 50%. No PV (night).
Omnibattery will discharge the battery at 500W to keep the P1 grid meter at 0W.

Now a big power user is switched on, it uses 4000W. Omnibattery will increase the battery discharge to 4500W to keep the
P1 grid meter at 0W. The SOC (State of Charge) falls quickly, 49%...45%...40%...36%...33%.

When the SOC reaches 32%, the "SOC Threshold", peak shaving starts. This means that Omnibattery will try to conserve 
energy (battery charge) just to make sure the monthly peak won't increase (and you have to pay more peak tarif).

Omnibattery will not try to regulate 0W on the P1 grid meter, but 2800W (3000-200, "P1 peak" - "Monthly Peak Offset"). 
The SOC keeps falling, but not as quickly as before. Omnibattery now discharges the battery at 1700W, much lower than 
the 4500W before with peak shaving.

When the SOC hits 30% (SOC Target), nothing happens at this moment. 

When the SOC hits 29% (SOC Floor), Omnibattery wants to recharge the battery. This is not possible at the moment 
because we are stil discharging the battery at 1700W. The SOC keeps falling.

At some point, the big power user is switched off. The power draw is now again 500W (home), SOC is at 22%. This is lower 
than the SOC Floor, and since peak shaving is active Omnibattery will start recharging the battery.

Omnibattery will draw 2700W from the grid (3000-300, "P1 peak" - "Charge Grid Offset"). The SOC will start to rise 
because we are charging the battery at 2200W (2700-500). SOC is 23%...26%...29%. 

At 30% SOC, SOC Target, charging of the battery will be stopped. Peak shaving is still active, so grid power draw is 
not 0W, but 500W (what the home uses). Omnibattery tries to conserve energy (battery charge).

Now the big power user is switched on again. Omnibattery discharges the battery again at 1700W to keep the power draw 
from the grid at 2800W. SOC falls 27%...20%..15%...13%.

At 12% SOC the battery is empty and stops discharging. Grid power draw is now 4500W, way above our month peak of 3000W.

This quarter hour ends, and the new P1 peak is 3628W. The peak limit value (omnibattery_capacity_protection_limit)
will be set to 3428 (3628-200).

As the next quarter hour ends, the P1 peak is now 4500W. The peak limit value will be set to 4300W (4500-200). 

When the big power user is switched off, the battery will be recharged again to 30% SOC.






help at:


Install Node-RED in HA: https://community.home-assistant.io/t/home-assistant-community-add-on-node-red/55023


Node-RED essentials : https://www.youtube.com/watch?v=ksGeUD26Mw0&list=PLyNBB9VCLmo1hyO-4fIZ08gqFcXBkHy-6

