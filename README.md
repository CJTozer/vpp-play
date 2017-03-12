Configuring VPP lives in `build_lb_config`.  Edit/run that to do the set up.

Run VPP with `run_vpp`.

Simple test lives in `test_lb`.  Needs a listener to prove that things are working...  (e.g. `nc -uvlk4 127.0.0.4 3000`)
