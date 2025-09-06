# Modern Linux Firewall with nftables 🛡️

A guide to configuring a secure, stateful firewall on a modern Linux system using **`nftables`**, the successor to `iptables`.

This configuration uses a single, declarative file (`/etc/nftables.conf`) to establish a "deny by default" policy, providing a strong security posture by blocking all unsolicited incoming traffic.

---

## Configuration Logic

The firewall operates on a few simple principles:

1.  **Unified Table:** A single `inet` table (`my_firewall`) is used to handle both IPv4 and IPv6 traffic simultaneously.
2.  **Default `DROP` Policy:** The `input` chain's default policy is set to **`DROP`**. This means any incoming packet that doesn't explicitly match an `accept` rule is silently discarded.
3.  **Stateful Inspection:** The firewall is stateful, using the `ct state established,related accept` rule. This is the core of the configuration, as it automatically allows return traffic for connections initiated by your machine, without needing to define rules for every possible response port.
4.  **Explicit `ACCEPT` Rules:** Only essential services and protocols are explicitly allowed through the `input` chain, including loopback traffic, ICMP (ping), SSH, and web traffic (HTTP/HTTPS).

---

## ⚙️ How to Use and Apply

Instead of a script, `nftables` is managed through its main configuration file.

1.  **Backup Your Existing Configuration**
    Before making any changes, it's critical to back up the current file:
    ```bash
    sudo cp /etc/nftables.conf /etc/nftables.conf.bak
    ```

2.  **Edit the `nftables.conf` File**
    Open the configuration file with a text editor like `nano`:
    ```bash
    sudo nano /etc/nftables.conf
    ```
    Delete all existing content and paste the following configuration:

    ```bash
    #!/usr/sbin/nft -f

    flush ruleset

    table inet my_firewall {
        chain input {
            type filter hook input priority 0;
            policy drop;

            # Allow loopback traffic and established/related connections
            iifname "lo" accept
            ct state established,related accept

            # Allow essential protocols
            ip protocol icmp accept      # Allow Ping

            # Allow specific service ports
            tcp dport 22 accept          # Allow SSH
            tcp dport 80 accept          # Allow HTTP
            tcp dport 443 accept         # Allow HTTPS
        }

        chain forward {
            type filter hook forward priority 0;
            policy drop;
        }

        chain output {
            type filter hook output priority 0;
            policy accept;
        }
    }
    ```

3.  **Apply the New Rules**
    To apply the configuration, restart the `nftables` service:
    ```bash
    sudo systemctl restart nftables
    ```

---

## ✅ Verification and Persistence

1.  **Verify the Rules**
    To see your new ruleset live, run:
    ```bash
    sudo nft list ruleset
    ```
    The output should match the contents of the file you just saved.

2.  **Ensure Persistence on Reboot**
    The `nftables` service automatically loads the `/etc/nftables.conf` file on boot. To ensure the service itself starts automatically, enable it with `systemctl`:
    ```bash
    sudo systemctl enable nftables
    ```
    Your firewall rules are now permanent and will survive a reboot.

---

## 🔧 Customization

To customize the firewall, simply edit the `/etc/nftables.conf` file and then restart the service.

* **To allow a new service:** Add a new rule to the `chain input` block. For example, to allow a service on port `3000`:
    ```bash
    # Inside the chain input { ... } block
    tcp dport 3000 accept
    ```
* **To block a service:** Remove or comment out its line using a `#`.

After any change, remember to apply it by restarting the service: `sudo systemctl restart nftables`.
