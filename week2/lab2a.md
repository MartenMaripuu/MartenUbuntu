[Unit]
Description=Append disk usage to log
Documentation=man:df(1)
After=local-fs.target

[Service]
Type=oneshot
User=reports
ExecStart=/usr/local/bin/disk-report.sh
StandardOutput=append:/var/log/disk-report.log
StandardError=append:/var/log/disk-report.log

[Unit]
Description=Run disk-report every five minutes
Documentation=systemd.time(7)

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target

Sep 22 06:00:04 ubuntu3 disk-report.sh[3017]: /usr/local/bin/disk-report.sh: line 2: /var/log/disk-report.log: Permission denied
Sep 22 06:00:04 ubuntu3 systemd[1]: disk-report.service: Main process exited, code=exited, status=1/FAILURE

The error means the reports user couldn't write to the log file. That's because I made the log file earlier while testing as root, so root owned it, not reports. Since reports is not root, it got blocked.

I could have just given the reports user permission with chown, but that means reports can write to that file forever, which is more access than it really needs. Instead I used StandardOutput=append: in the service file. This lets systemd (which runs as root) write the log for the script, so the reports user never needs write access at all. It's safer because reports stays low permission the whole time.

NEXT: Wed 2026-09-23 00:00:00 UTC
LAST: Tue 2026-09-22 06:00:04 UTC
1 timers listed.

5. Two successful runs

Sep 22 06:02:54 ubuntu3 systemd[1]: Finished disk-report.service - Append disk usage to log.
Sep 22 06:04:46 ubuntu3 systemd[1]: Finished disk-report.service - Append disk usage to log.

The reports user was made with --no-create-home and --shell /usr/sbin/nologin so nobody can actually log into it like a real account. If those weren't there, someone who hacked the reports account could log in and use it like a normal user, which would let them do a lot more damage than just running one script.


