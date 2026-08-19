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

If someone asks:

"Application is crashing. What command will you use?"

A strong answer is:

tail -f /var/log/app.log

because you can watch new log entries in real time while reproducing the issue.

Q3:

A production server is:

CPU usage is normal
Memory is normal
Disk is normal
But application is still slow
👉 What will you check next?

ANS:

✔️ Step-by-step thinking
1. Check application logs
tail -f /var/log/app.log

Look for:

errors
slow queries
timeouts
2. Check network latency / connectivity
ping <service>
curl -v http://localhost:port
3. Check dependency services
Database slow?
Redis down?
External API slow?

Example:

mysql -h db-host -u user -p
4. Check thread / process state
top -H

or

ps -ef | grep java

Look for:

blocked threads
high IO wait
deadlocks
5. Check disk IO latency (important)
iostat -x 1

Even if disk is NOT full, it can still be slow


