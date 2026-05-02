# Networking & Git Q&A (100+ Questions)

Basic, Advanced, and Scenario-based questions.

---

# Networking (70 Questions)

## Basic

**Q1: What is an IP address?**
A: Unique identifier for a device on a network. IPv4 (32-bit) or IPv6 (128-bit).

**Q2: What is IPv4 vs IPv6?**
A: IPv4 = 32-bit, ~4 billion addresses (e.g., 192.168.1.1). IPv6 = 128-bit, virtually unlimited (e.g., 2001:db8::1).

**Q3: What is a private IP address?**
A: Not routable on internet. 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16.

**Q4: What is a public IP address?**
A: Routable on internet. Assigned by ISP or cloud provider.

**Q5: What is NAT?**
A: Network Address Translation. Maps private IPs to public IP. Allows multiple devices to share one public IP.

**Q6: What is a subnet?**
A: Subdivision of a network. Defined by subnet mask.

**Q7: What is CIDR notation?**
A: Classless Inter-Domain Routing. IP/prefix (e.g., 10.0.0.0/24 = 256 IPs).

**Q8: What does /24 mean?**
A: 24 bits for network, 8 bits for hosts. 256 IP addresses (254 usable).

**Q9: What is a subnet mask?**
A: Defines network vs host portion. 255.255.255.0 = /24.

**Q10: What is a gateway?**
A: Router that connects networks. Default gateway routes traffic outside local network.

**Q11: What is DNS?**
A: Domain Name System. Translates domain names to IP addresses.

**Q12: What is a DNS record?**
A: Entry in DNS. A (IPv4), AAAA (IPv6), CNAME (alias), MX (mail), TXT (text).

**Q13: What is an A record?**
A: Maps domain name to IPv4 address.

**Q14: What is a CNAME record?**
A: Alias pointing to another domain name.

**Q15: What is TTL in DNS?**
A: Time To Live. How long to cache the record.

**Q16: What is a port?**
A: Logical endpoint for communication. 0-65535. Well-known: 0-1023.

**Q17: What are common ports?**
A: 22 (SSH), 80 (HTTP), 443 (HTTPS), 3306 (MySQL), 5432 (PostgreSQL), 6379 (Redis).

**Q18: What is TCP?**
A: Transmission Control Protocol. Connection-oriented, reliable, ordered delivery.

**Q19: What is UDP?**
A: User Datagram Protocol. Connectionless, fast, no guarantee of delivery.

**Q20: When would you use UDP over TCP?**
A: Real-time applications (video, gaming, DNS queries) where speed matters more than reliability.

**Q21: What is the TCP three-way handshake?**
A: SYN → SYN-ACK → ACK. Establishes connection.

**Q22: What is HTTP?**
A: Hypertext Transfer Protocol. Application layer protocol for web. Stateless.

**Q23: What is HTTPS?**
A: HTTP over TLS/SSL. Encrypted communication.

**Q24: What is TLS/SSL?**
A: Transport Layer Security / Secure Sockets Layer. Encrypts data in transit.

**Q25: What is a certificate?**
A: Digital document proving identity. Contains public key. Issued by CA.

**Q26: What is a CA?**
A: Certificate Authority. Trusted entity that issues certificates.

**Q27: What is the OSI model?**
A: 7-layer networking model. Physical, Data Link, Network, Transport, Session, Presentation, Application.

**Q28: What layer is IP?**
A: Layer 3 (Network).

**Q29: What layer is TCP/UDP?**
A: Layer 4 (Transport).

**Q30: What layer is HTTP?**
A: Layer 7 (Application).

**Q31: What is a MAC address?**
A: Media Access Control. Hardware address of network interface. 48-bit.

**Q32: What is ARP?**
A: Address Resolution Protocol. Maps IP addresses to MAC addresses.

**Q33: What is DHCP?**
A: Dynamic Host Configuration Protocol. Automatically assigns IP addresses.

**Q34: What is a firewall?**
A: Controls network traffic based on rules. Allows or blocks connections.

**Q35: What is a load balancer?**
A: Distributes traffic across multiple servers. Improves availability and performance.

---

## Advanced

**Q36: What is a VLAN?**
A: Virtual LAN. Logically separates network at Layer 2.

**Q37: What is a VPN?**
A: Virtual Private Network. Encrypted tunnel over public network.

**Q38: What is IPSec?**
A: Protocol suite for securing IP communications. Used in VPNs.

**Q39: What is BGP?**
A: Border Gateway Protocol. Routes traffic between autonomous systems (internet backbone).

**Q40: What is OSPF?**
A: Open Shortest Path First. Interior routing protocol.

**Q41: What is a CDN?**
A: Content Delivery Network. Caches content at edge locations globally.

**Q42: What is latency?**
A: Time for data to travel from source to destination. Measured in milliseconds.

**Q43: What is bandwidth?**
A: Maximum data transfer rate. Measured in Mbps/Gbps.

**Q44: What is throughput?**
A: Actual data transfer rate achieved.

**Q45: What is jitter?**
A: Variation in latency. Important for real-time applications.

**Q46: What is packet loss?**
A: Percentage of packets that don't reach destination.

**Q47: What is MTU?**
A: Maximum Transmission Unit. Largest packet size without fragmentation. Usually 1500 bytes.

**Q48: What is a proxy?**
A: Intermediary between client and server. Forward proxy (client-side) or reverse proxy (server-side).

**Q49: What is a reverse proxy?**
A: Sits in front of servers. Load balancing, caching, SSL termination.

**Q50: What is SNI?**
A: Server Name Indication. TLS extension allowing multiple certificates on one IP.

**Q51: What is a socket?**
A: Endpoint for communication. IP address + port.

**Q52: What is connection pooling?**
A: Reusing connections instead of creating new ones. Reduces overhead.

**Q53: What is keep-alive?**
A: Maintaining connection for multiple requests. Reduces latency.

**Q54: What is a DDoS attack?**
A: Distributed Denial of Service. Overwhelming target with traffic.

**Q55: What is rate limiting?**
A: Restricting number of requests per time period. Prevents abuse.

---

## Scenario-Based

**Q56: You can't reach a website. How do you troubleshoot?**
A:
1. `ping domain` - Is it reachable?
2. `nslookup domain` - DNS resolving?
3. `traceroute domain` - Where does it fail?
4. `curl -v domain` - HTTP level issues?
5. Check firewall, proxy settings

**Q57: A server is slow to respond. What do you check?**
A: Network latency (ping), bandwidth saturation, packet loss, server load, application performance.

**Q58: How do you find what's listening on a port?**
A: `ss -tlnp | grep :80` or `lsof -i :80` or `netstat -tlnp | grep :80`

**Q59: How do you test if a port is open?**
A: `telnet host port`, `nc -zv host port`, or `curl host:port`

**Q60: How do you capture network traffic?**
A: `tcpdump -i eth0 port 80` or Wireshark.

**Q61: How do you check DNS resolution?**
A: `nslookup domain`, `dig domain`, `host domain`

**Q62: How do you flush DNS cache?**
A: Linux: `systemd-resolve --flush-caches`. Mac: `sudo dscacheutil -flushcache`.

**Q63: How do you test SSL certificate?**
A: `openssl s_client -connect host:443` or `curl -vI https://host`

**Q64: A connection times out. What could be wrong?**
A: Firewall blocking, wrong IP/port, service not running, network routing issue.

**Q65: How do you check network interface configuration?**
A: `ip addr` or `ifconfig`

**Q66: How do you add a static route?**
A: `ip route add 10.0.0.0/8 via 192.168.1.1`

**Q67: How do you check current connections?**
A: `ss -tunapl` or `netstat -tunapl`

**Q68: What causes "connection refused"?**
A: Service not running on that port, or firewall rejecting connection.

**Q69: What causes "no route to host"?**
A: Network unreachable, routing misconfiguration.

**Q70: How do you test bandwidth?**
A: `iperf3` between two hosts.

---

# Git (50 Questions)

## Basic

**Q71: What is Git?**
A: Distributed version control system. Tracks changes to files.

**Q72: What is a repository?**
A: Project folder tracked by Git. Contains .git directory.

**Q73: What is a commit?**
A: Snapshot of changes. Has unique SHA hash.

**Q74: What is a branch?**
A: Pointer to a commit. Allows parallel development.

**Q75: What is the main/master branch?**
A: Default primary branch. Usually represents production code.

**Q76: How do you create a new branch?**
A: `git branch branch-name` or `git checkout -b branch-name`

**Q77: How do you switch branches?**
A: `git checkout branch-name` or `git switch branch-name`

**Q78: How do you merge branches?**
A: `git checkout main` then `git merge feature-branch`

**Q79: What is a merge conflict?**
A: When same lines changed in both branches. Must be resolved manually.

**Q80: How do you resolve merge conflicts?**
A: Edit conflicted files, remove conflict markers, stage and commit.

**Q81: What is git pull?**
A: Fetches and merges remote changes. `git fetch` + `git merge`.

**Q82: What is git push?**
A: Uploads local commits to remote repository.

**Q83: What is git fetch?**
A: Downloads remote changes without merging.

**Q84: What is a remote?**
A: Reference to remote repository. Usually "origin".

**Q85: What is git clone?**
A: Creates local copy of remote repository.

**Q86: What is git status?**
A: Shows working directory and staging area status.

**Q87: What is the staging area?**
A: Intermediate area before commit. `git add` stages changes.

**Q88: What is git diff?**
A: Shows differences between commits, branches, or working directory.

**Q89: What is git log?**
A: Shows commit history.

**Q90: What is .gitignore?**
A: Specifies files Git should ignore. Build artifacts, secrets, etc.

---

## Advanced

**Q91: What is git rebase?**
A: Moves commits to new base. Creates linear history. Alternative to merge.

**Q92: When would you use rebase vs merge?**
A: Rebase for clean history (feature branches). Merge for preserving history (shared branches).

**Q93: What is git cherry-pick?**
A: Applies specific commit to current branch.

**Q94: What is git stash?**
A: Temporarily saves uncommitted changes. `git stash`, `git stash pop`.

**Q95: What is git reset?**
A: Moves HEAD and optionally changes staging/working directory. --soft, --mixed, --hard.

**Q96: What's the difference between reset and revert?**
A: Reset changes history (dangerous for shared branches). Revert creates new commit undoing changes (safe).

**Q97: What is git reflog?**
A: Log of all HEAD movements. Helps recover lost commits.

**Q98: What is a tag?**
A: Named reference to specific commit. For releases. `git tag v1.0.0`

**Q99: What is git bisect?**
A: Binary search to find commit that introduced bug.

**Q100: What is a submodule?**
A: Repository inside another repository. For dependencies.

**Q101: What is git blame?**
A: Shows who last modified each line of a file.

**Q102: What is a squash merge?**
A: Combines all commits into one when merging. Cleaner history.

**Q103: What is a fast-forward merge?**
A: When target branch hasn't diverged. Just moves pointer. No merge commit.

**Q104: What is HEAD?**
A: Pointer to current commit. Usually points to branch tip.

**Q105: What is detached HEAD?**
A: HEAD pointing directly to commit, not branch. Commits won't belong to any branch.

---

## Scenario-Based

**Q106: You committed to wrong branch. How do you fix it?**
A: `git cherry-pick` to correct branch, then `git reset --hard HEAD~1` on wrong branch.

**Q107: You need to undo last commit but keep changes.**
A: `git reset --soft HEAD~1`

**Q108: You need to completely undo last commit.**
A: `git reset --hard HEAD~1` (local only) or `git revert HEAD` (if pushed).

**Q109: You accidentally deleted a branch. How do you recover?**
A: `git reflog` to find commit, `git checkout -b branch-name commit-sha`.

**Q110: How do you update a feature branch with main?**
A: `git checkout feature` then `git merge main` or `git rebase main`.

**Q111: How do you see what changed in a commit?**
A: `git show commit-sha` or `git diff commit-sha^..commit-sha`

**Q112: How do you find which commit introduced a bug?**
A: `git bisect start`, `git bisect bad`, `git bisect good commit`, then test each.

**Q113: How do you amend the last commit message?**
A: `git commit --amend -m "new message"` (only if not pushed).

**Q114: How do you add forgotten file to last commit?**
A: `git add file` then `git commit --amend --no-edit`.

**Q115: How do you clean up local branches?**
A: `git branch -d branch-name` (merged) or `git branch -D branch-name` (force).

**Q116: How do you see remote branches?**
A: `git branch -r` or `git branch -a` (all).

**Q117: How do you delete a remote branch?**
A: `git push origin --delete branch-name`

**Q118: How do you rename a branch?**
A: `git branch -m old-name new-name`

**Q119: How do you create a PR/MR from command line?**
A: `gh pr create` (GitHub CLI) or `glab mr create` (GitLab CLI).

**Q120: How do you sign commits?**
A: `git commit -S` with GPG key configured.

---

*Master networking and Git — they're fundamental to everything in DevOps! 🌐*
