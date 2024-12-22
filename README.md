# Handle.TPM

Seals secrets within the TPM

## Installation

On Arch-Based Distributions, download the PKGBUILD and run `makepkg -si`

For others, ensure that `systemd` `tpm2-tools` `openssl` `coreutils` `bash` are installed, and run `chmod +x` on the script.

## Usage

`handle.tpm` has three modes of operation, defined by the `-m` argument:

1. `seal`: Seal a secret at the provided TPM address.
2. `unseal`: Unseal and return the secret at the provided TPM address on STDOUT.
3. `evict`: Remove a secret at the provided TPM address.
4. `erase`: Overwrite the PCR values.

The address is provided by the `-a` argument.

By default, `handle.tpm` will seal secrets using PCR registers 0+7, and will overwrite these registers upon unsealing. This allows for the script to be used to decrypt a boot volume, and then overwrite the registers so that the booted system cannot retrieve the keys. Such a use can be seen in `strongbox-tpm2`.

If you are unsealing multiple passwords during boot, such as for multiple devices, you can pass the `-p` to preserve the PCR state. This is very dangerous, as failing to randomize the values allows for the booted computer to trivially access the secrets. You should always wipe the PCRs by omitting this argument when the last device is being unsealed.

> [!warning]
> `handle.tpm` wipes the PCRs after *every* transaction in which the `-p` flag is not supplied, even transactions that do not involve the TPM. If you are enrolling/unsealing multiple secrets from a trusted state, make sure that the `-p` flag is in use with each one, and then call `handle-tpm -m erase` to wipe.

However, `handle.tpm` can also be used to store secrets without sealing it using the PCRs. Without any way to ensure a controlled environment, however, sealing the secrets in this way would usually make them trivial to retrieve, only requiring root access. For this reason, `handle.tpm` will require a sealing pin to be provided, which will then be used to encrypt the secret using `openssl`. Such use can be seen in `safe`.

To disable sealing with the PCRs, provide the `-n` argument. To define the iteration rounds that `openssl` uses for encryption, provide the `-i ROUNDS` argument (If you explicitly define a rounds, you will need to provide it both during encryption and decryption).

Finally, a secret can be evicted using the `evict` mode, which frees up space to be used for another secret.

By default, input is obtained using `systemd-ask-password`, which permits an interact user entry that works in an initramfs environment (Such as with plymouth). To read from standard input instead, use the `-s` switch.

To use an enhanced seal, combining TPM attestation with a sealing PIN, use the `-e` switch.

> [!warning]
> Secrets sealed using `handle.tpm` *cannot* be retrieved in situations of TPM errors or resets. The design of `handle.tpm` is specifically for storing secrets that you are able to remember, because failure of the TPM can lead to irreovable loss. Sealing pins, cannot not be used in place of the original keys if the TPM is faulty or reset, and should be thought of as additional, simpler passwords. If you are storing secrets you cannot type or remember, such as random keys, understand that any data relying on that key can be irrevocably lost if you do not make a backup copy! Use `handle.tpm` to store complex passwords that the TPM can then replace either entirely in a trusted state, or replace with a simpler PIN, but that you can still type in should issues arise!