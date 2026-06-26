Q1:
You're logged into a Linux production server.

A developer says:

"The application is down."

What are the first 8-10 commands you would run to diagnose the issue?

Don't just list commands—tell me:

the command,
why you're running it,
and what you're expecting to find.

ANS:

uptime – Check server load and uptime.
df -h – Check disk usage.
free -h – Check available RAM.
top or htop – Check CPU and memory consumption.
ps -ef | grep <app> – Verify the application process is running.
systemctl status <service> – If it's managed as a systemd service.
docker ps or kubectl get pods – Check container/pod status.
docker logs <container> or kubectl logs <pod> – Inspect application logs.
kubectl describe pod <pod> – Investigate events like CrashLoopBackOff, image pull failures, etc.

Q2:
