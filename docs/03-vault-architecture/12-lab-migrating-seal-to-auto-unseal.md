# Lab: Migrating Seal to Auto Unseal

* [Take me to the lab](https://kodekloud.com/topic/lab-migrating-seal-to-auto-unseal-3/)
* [Help for VSCode editor](https://github.com/kodekloudhub/community-faq/blob/main/docs/vscode-tips.md)

1.  Information only

1.  <details>
    <summary>What is the status of the Vault cluster?</summary>

    * Unsealed and not initialized
    * Unsealed and initialized
    * Sealed and initialized
    * Sealed and not initialized

    <details>
    <summary>Reveal</summary>

    Run the command: `vault status` and check the Initialized and Sealed values.

    > Sealed and not initialized

    </details>
    </details>

1.  <details>
    <summary>Let's take a look at the Vault configuration file which is located in the <code>/etc/vault.d</code> directory. Does it have a seal configuration?</summary>

    * Yes
    * No

    <details>
    <summary>Reveal</summary>

    1. Check files in this directory

        ```bash
        ls -l /etc/vault.d
        ```
    2. Note the config file `vault.hcl`. Examine its contents

        ```bash
        cat /etc/vault.d/vault.hcl
        ```

    3. Note that there is no mention of seal in there, so the answer is...

    > No

    </details>
    </details>

1.  <details>
    <summary>Let's go ahead and initialize our Vault node but reduce the number of key shares to 3 and the key thresholds to 2.</summary>

    As per the below format, store all three unseal keys in the `/root/unseal_keys` file. Example: -

    ```
    Unseal Key 1: <KEY-1>
    Unseal Key 2: <KEY-2>
    Unseal Key 3: <KEY-3>
    ```

    and the root token in the `/root/main_token` file. Example: -

    ```
    Initial Root Token: <TOKEN>
    ```
    <details>
    <summary>Reveal</summary>

    1. Run the command:

        ```bash
        vault operator init -key-shares=3 -key-threshold=2
        ```

    1. In the VSCode explorer pane, right click to create files. The explorer is positioned on `/root` directory, so you can name them `unseal_keys` and `main_token`

    1. Copy from the console output of the command you ran, the unseal keys and token to the appropriate files.

    </details>
    </details>

1.  <details>
    <summary>Unseal the Vault node using 2 of the unseal keys from the Vault initialization which we stored in the <code>/root/unseal_keys</code> file.</summary>

    * Use the `vault operator unseal` command to unseal Vault.

    <details>
    <summary>Reveal</summary>

    You'll need to run the above command twice since we set the key threshold to 2. At each invocation, use a different key from the unseal keys generated in the previous question.

    After running it for the second time, the output should indicate that it's unsealed and initiailized...

    ```
    Unseal Key (will be hidden):
    Key                     Value
    ---                     -----
    Seal Type               shamir
    Initialized             true
    Sealed                  false
    ```

    </details>
    </details>

1.  <details>
    <summary>Log in to Vault using the root token and enable a new KV V2 secrets engine at the path of <code>secrets/</code>.</summary>

    * Use the `vault login` and `vault secrets` commands.

    <details>
    <summary>Reveal</summary>

    1. Using the token created in Q4, log into Vault

        ```bash
        vault login <YOUR_TOKEN>
        ```

    1. Create the secrets engine

        ```bash
        vault secrets enable -path=secrets kv-v2
        ```

    </details>
    </details>

1.  <details>
    <summary>Now, restart the Vault service and validate the sealed state.</summary>

    * Sealed
    * Unsealed

    <details>
    <summary>Reveal</summary>

    1. Run

        ```bash
        systemctl restart vault
        vault status
        ```

    > Sealed

    It is in a sealed state. This is the default behavior when using the default seal configuration of unseal keys.
    </details>
    </details>

1.  <details>
    <summary>In the upcoming questions, we will make use of the awskms seal stanza for auto unseal. Before going ahead and stop the Vault service so we can make some changes.</summary>

    ```bash
    systemctl stop vault
    ```

1.  <details>
    <summary>Now, let's reconfigure our Vault node so we can migrate from the default seal mechanism to use the AWS auto unseal instead. This will allow Vault to automatically unseal itself when the service is restarted.</summary>

    In this task, we're going to use an AWS KMS key for auto unseal. We have already created a key for you and are available in the /root/kms_key file.


    Add the below seal stanza to the Vault configuration file: -

    ```hcl
    seal "awskms" {
        region = "us-east-1"
        kms_key_id = "<Enter KMS ARN here>"
    }
    ```
    and don't forget to update the kms_key_id value.


    **NOTE**: - Don't need to start the Vault service as of now. We will start later in this lab.

    <details>
    <summary>Reveal</summary>

    1. Add the `/etc/vault.d` folder to VSCode explorer pane so that you can edit the files using VSCode. See [here](https://github.com/kodekloudhub/community-faq/blob/main/docs/vscode-tips.md#how-to-add-folders-to-the-workspace) for how to do this.

    1. Select the `vault.hcl` file in the explorer pane to open it.
    1. Copy the `seal` stanza above and paste at the end of the file.
    1. In VSCode editor, select `kms_key` in the explorer pane to open the file, and copy the key.
    1. Return to `vault.hcl` and edit out `<Enter KMS ARN here>` and paste in the kms key in its place.

    </details>
    </details>

1.  <details>
    <summary>Before going to start the Vault service. We need to provide Vault with AWS credentials to access and use the KMS key.</summary>

    1. Select the `vault.env` file in the explorer pane to open it.
    1. Copy and paste in the 3 AWS environment variables from the question. They may be different each time you run the lab.

1.  <details>
    <summary>Start the Vault service and check the status of the service to ensure it's running.</summary>

    Run these commands

    ```bash
    systemctl start vault
    systemctl status vault
    ```

1.  <details>
    <summary>Did Vault unseal itself automatically?</summary>

    Check the status of Vault itself (not the service)

    <details>
    <summary>Reveal</summary>

    ```bash
    vault status
    ```

    > No

    </details>
    </details>

1.  <details>
    <summary>Let's perform the seal migration. Use the 2 unseal keys we received from our original Vault initialization.</summary>

    * Use the `vault operator unseal` command with appropriate argument.


    <details>
    <summary>Reveal</summary>

    Return to the `unseal_keys` file in VSCode. Ensure to use the same 2 keys as for the initial unseal at the beginning of this lab. Again, the command must be run twice - one for each key

    ```bash
    vault operator unseal -migrate
    ```

    </details>
    </details>

1.  <details>
    <summary>Observe the Vault server is functional and validate the sealed state.</summary>

    The state should be clear from the output of the commands you ran in the last question

    <details>
    <summary>Reveal</summary>

    > Unsealed

    </details>
    </details>

1.  <details>
    <summary>Log in to the Vault using the Root token and answer the below question.</summary>

    * How many secret engines are enabled by default?

    <details>
    <summary>Reveal</summary>

    1. Log in with token as you did in Q6
    1. Run the following and count the secrets engines
        ```bash
        vault secrets list
        ```

    > 4

    </details>
    </details>

1.  <details>
    <summary>After the successful migration to auto unseal, Vault should be automatically unsealed after a service restart.</summary>

    Restart the service and check the status of the Vault.

    Is Vault unsealed?

    <details>
    <summary>Reveal</summary>

    Run the following commands

    ```bash
    systemctl restart vault
    ```

    Wait a few seconds, then

    ```
    vault status
    ```

    > Yes

    </details>
    </details>

1.  <details>
    <summary>Let's log in to Vault using the root token and try to list the secrets engines that are already enabled. It should work without any issues.</summary>

    Repeat the commands you ran in Q15

    </details>
    </details>
