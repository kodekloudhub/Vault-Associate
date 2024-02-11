# Lab: Vault Configuration File

* [Take me to the lab](https://kodekloud.com/topic/lab-vault-configuration-file-3/)
* [Help for VSCode editor](https://github.com/kodekloudhub/community-faq/blob/main/docs/vscode-tips.md)

1.  Information only

1.  <details>
    <summary>Vault configuration files are written in?</summary>

    * HCL
    * JSON
    * None of these
    * Both HCL and JSON

    <details>
    <summary>Reveal</summary>

    > Both HCL and JSON

    The configuration can be provided in either format, but HCL is preferred. Before HCL was developed, configuration for all Hashicorp products was done in JSON.

    </details>
    </details>

1.  <details>
    <summary><code>HCL</code> stands for?</summary>

    * Hashicorp Code Language
    * High-level Compiled Language
    * Hashicorp Configuration Language
    * High Configuration Language

    <details>
    <summary>Reveal</summary>

    > Hashicorp Configuration Language

    HCL is used in many of Hashicorp's products, including Vault, Terraform and Packer.

    </details>
    </details>

1.  <details>
    <summary>Let's get started to write the vault configuration file. We have already created an empty <code>vault.hcl</code> configuration file which is available at <code>/etc/vault.d/</code> directory and open this configuration file through the code editor.</summary>

    And write the following parameters as follows: -

    * Type of the Storage Backend: `Integrated Storage(Raft)`
    * Path for the data storage: `/opt/vault/data`
    * Node ID for this server would be: `vault-node-a`


    <details>
    <summary>Reveal</summary>

    1. Add the `/etc/vault.d` folder to VSCode explorer pane so that you can edit the files using VSCode. See [here](https://github.com/kodekloudhub/community-faq/blob/main/docs/vscode-tips.md#how-to-add-folders-to-the-workspace) for how to do this.
    1. Add the storage sanza for `raft`
    1. Add the other two parameters within the stanza

    The result shuod look like this

    ```hcl
    storage "raft" {
      path    = "/opt/vault/data"
      node_id = "vault-node-a"
    }
    ```

    </details>
    </details>

1.  <details>
    <summary>What is the ONLY valid type of listener stanza supported by Vault?</summary>

    * tcp
    * udp
    * icmp
    * any

    <details>
    <summary>Reveal</summary>

    > tcp

    * `tcp` guarantees delivery of a message to a remote peer, or reports an error so that the sender may retry. Thus Vault listens on tcp such that vault clients (the `vault` CLI or user interface) can know that the commands were successfully received by the server.
    * `udp` does not guarantee delivery of messages to a remote peer and does not report transmission errors back to the sender. As such is used for things like video streaming protocols where a few transmission errors can be tolerated. For a secure secrets management platform, no errors can be tolerated.
    * `icmp` is not a protocol for sending meaningful data. It is used only for testing connectivity between endpoints, and is used by programs such as `ping`.
    * `any` is not an option. The clue's in the question... "ONLY".

    </details>
    </details>

1.  <details>
    <summary>If you wanted to configure this node to listen for client requests on ALL interfaces. Which configuration would allow you to do that within the listener stanza?</summary>

    * address = "vault.gswhv.com:443"
    * address = "0.0.0.0:8200"
    * address = "127.0.0.1:8200"
    * address = "10.x.x.x:8500"

    <details>
    <summary>Reveal</summary>

    > address = "0.0.0.0:8200"

    In this question, you can ignore the port number since this is about selecting the network interfaces that the server listens on.

    * `0.0.0.0:8200` is correct because `0.0.0.0` is how you refer to all intefaces when configuring a listener for anything, not just Vault.
    * The first option is invalid as you cannot use DNS names for selecting interfaces.
    * The third option is invailid, as that selects specifically the `localhost` interface which is not all interfaces.
    * The fourth option is invalid because you must use a vaild IP format, which is composed entirely of numbers.

    </details>
    </details>

1.  <details>
    <summary>Now, let's add the listener stanza to the same vault.hcl configuration file which is located at /etc/vault.d directory.</summary>

    Make use of the following parameters:

    * Address: - This should be listening on all local IP addresses using the default port of the API.
    * Cluster Address: - This should listen on a local IP address using the default port for RPC.
    * TLS: - This should be in disable mode.

    <details>
    <summary>Reveal</summary>

    Add a new stanza for the listener to the end of the `vault.hcl` file, similarly to how you did it in Q4. The finished product sholud look like this

    ```hcl
    listener "tcp" {
      address = "0.0.0.0:8200"
      cluster_address = "0.0.0.0:8201"
      tls_disable = 1
    }
    ```

    </details>
    </details>

1.  <details>
    <summary>Now let's configure how Vault will handle seal and unseal operations. Which of the following configurations will configure vault to split the <code>Root</code> Key into multiple key shares called <code>unseal</code> keys?</summary>

    * seal "transit" {}
    * no configuraton
    * seal "shamir" {}
    * seal "aws" {}

    <details>
    <summary>Reveal</summary>

    > no configuration

    Vault will use the default handler when no configuration is present, which is actually "shamir". This does not require any custom settings so a stanza isn't required.

    </details>
    </details>

1.  <details>
    <summary>Let's add more configuration to the <code>vault.hcl</code> configuration file that we have learned so far. Configuration file is available at <code>/etc/vault.d/</code> path.</summary>

    Make use of the following points to add configuration in the vault.hcl file: -
    Enable the UI so it can be accessed through the browser

    * Set the Log level to `ERROR`
    * Configure the cluster name to be `my-vault-cluster`
    * Use `vault.gswhv.com:8200` for the primary API address
    * Set the cluster address to use the node name (same domain) with the default port for RPC/replication

    <details>
    <summary>Reveal</summary>

    All of these settings are "variable value" parameters and do not go inside a stanza, so they are simply set as `variable = value` in the configuration. You should add the following at the end of the file.

    ```hcl
    ui = true
    log_level = "ERROR"
    api_addr = "http://vault.gswhv.com:8200"
    cluster_name = "my-vault-cluster"
    cluster_addr = "https://vault-node-a.gswhv.com:8201"
    ```

    </details>
    </details>

1.  Information only.