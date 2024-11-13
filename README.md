# Houdini
My tests in Houdini

## scales_ip_test
This is a solution for post-fixes on scales and feather IPs based on a simple mathematical method relayed to me by a rigging supervisor who was working at the same company as I was at the time. He had a successful implementation working in Maya and I had translated it into Houdini using VEX. So far I've recreated the solution only with a single row of scales, and it's a very robust solution for that particular problem. However, on a larger set of scales it requires more consider for more axes of rotation due to all the different angles of intersections that the scales can have. I had previously implemented a solution that worked relatively well, but not as well as I had originally intended it to be. The next step for this tool is to attempt to do something similar to that but in Python instead.

Original scales with IPs:

After fixing:
