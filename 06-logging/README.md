This manifest deploys pod that contains a busybox image. The container uses a simple script to increment a counter every second. It then writes a counter value to a log file called server.log.

1. Create a sidecar container that outputs the server.log file to stdout.