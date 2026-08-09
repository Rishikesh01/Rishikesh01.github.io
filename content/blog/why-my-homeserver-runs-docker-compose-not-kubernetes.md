---
title: "Why my homeserver runs Docker Compose, not Kubernetes"
date: 2026-08-09
description: "I had k0s installed and still ended up on plain Docker Compose. This is why."
---

I had Kubernetes installed on my homeserver. I still chose Docker Compose.

Not because Kubernetes couldn't run it. It absolutely could. I chose Compose
because the entire point of this server was to make my life easier, and I know
exactly where "I should probably just use Kubernetes" ends for me.

Let me back up.

I always wanted to have a server of my own. The list of what I needed was pretty
basic:

- **Cloud storage** ([Nextcloud](https://nextcloud.com/))
- **A password manager** ([Vaultwarden](https://github.com/dani-garcia/vaultwarden),
  works with the normal Bitwarden apps)

That was already enough to justify having a server. But I also wanted PDF and
image tools. If you live in India you know the drill with government websites:
compress this PDF under some size limit, crop that photo, resize this image to
exactly some KB. I was done doing that on random online converter sites, so
[Stirling-PDF](https://github.com/Stirling-Tools/Stirling-PDF) and a small
in-browser image tool went on the list too.

## HTTPS, even at home

I didn't want "it's only on my LAN" to be an excuse for running everything over
HTTP. All kinds of devices connect to this network, and I don't fully control
what's running on every one of them. [Caddy](https://caddyserver.com/) made
in-house HTTPS easy: it runs its own CA, and once a device trusts that root
certificate, everything on the server is proper HTTPS inside the house. Install
the cert once per device, and the padlock is genuine.

## My parents have to be able to use it

I also wanted my parents to use this thing. They're not going to remember IP
addresses and port numbers, so I needed one entry point: a simple website on the
server that shows them where everything is.

## k0s vs Proxmox vs Docker Compose

So that was the list. Then I had to decide how to actually run all of it. I
already had [k0s](https://k0sproject.io/) installed, I'd seen YouTubers run their
homeservers on Proxmox, and there was always plain Docker Compose.

Proxmox was the first one out. Why, you ask? My machine is an AMD Ryzen 3 4300U
with 4 cores. Running a hypervisor on that just to host a handful of containers
made no sense.

So it came down to k0s vs Docker Compose, and I was leaning towards k0s. Hey,
**KUBERNETES STANDALONE SOUNDS FUN.** I could mess around with things, go
multi-node, move workloads around, add ingress, alerting, all of it.

Docker Compose was boring. It had nothing going for it except being dead simple.

Then it hit me that dead simple is the whole point.

I know me playing with k0s on the homeserver is going to put me investigating an
incident at some unwanted time in the future. That's the opposite of what I want
from it. The homeserver is supposed to make my life easy.

Backups too. I picked [restic](https://restic.net): snapshots are encrypted and
deduplicated, so wherever they land only ever sees encrypted data. With Compose,
a service is basically its config directory and its volumes. To restore,
you put the files back and run `docker compose up -d`. I can do that half asleep
at 11pm. The same story in Kubernetes is manifests, persistent volumes and cluster
state. No thanks.

It was Kubernetes being fun for me, or the server always working and remaining
dead simple to maintain.

## So where did the fun go?

I picked boring: one directory per service, Caddy in front, restic for backups,
and `hsctl backup verify` to prove restores actually work, because I don't trust
a backup I've never restored.

But I didn't stop tinkering. I moved the complexity out of the infrastructure and
into tooling I control: [`hsctl`](https://github.com/Rishikesh01/homeserver), the
Go binary that sets everything up, runs it, backs it up, and serves the dashboard
my parents use. If hsctl breaks, the services keep running. The infrastructure
stays boring, and the fun lives in a layer that can't take the server down with
it.

Everything on that dashboard exists because something kept annoying me:

**The terminal.** I was tired of sshing into the server for every little thing.
Start a backup, run an `apt install`, anything that needs root. Now I open the
browser, click Terminal, and I have a root shell from any device in the house.
It's LAN-only, behind Caddy's HTTPS and a login, so it's not as crazy as "root
shell in the browser" sounds.

**The sandbox.** If I ever need to upgrade restic, I want to test that restore
still works without nuking my existing data. `make sandbox` boots the whole stack
inside docker-in-docker with my real backup repo mounted read-only. I can do a
full restore in there, log into the restored Vaultwarden and Nextcloud, poke at my
actual data, and throw it all away after. The real server never notices.

**Disk health.** An SSD won't warn me it's about to die unless I actively go check
SMART, and I'm not going to remember to run `smartctl` on a schedule. So the
dashboard just shows drive health where I'll actually see it.

**The mount button.** Mounting an external HDD meant ssh, `lsblk`, figure out
which device it is this time, type the mount command by hand. It was a pain every
single time. Now there's a tab in admin where I pick the disk and hit mount. Done.
That killed the last regular reason I had to ssh into the box at all.

Most of that dashboard code was built with Claude Code. I designed the features,
decided what belonged in the system, and, more importantly, tested the hell out of
it. The sandbox exists partly because I don't trust software just because it
compiled.

The whole setup is open source: [github.com/Rishikesh01/homeserver](https://github.com/Rishikesh01/homeserver).

The infrastructure is boring. That's exactly how I wanted it.
