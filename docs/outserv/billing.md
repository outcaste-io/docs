---
sidebar_position: 2
---

# 2. Pay for Outserv

:::note
Outserv is licensed under the [Sustainable
License](https://manishrjain.com/sustainable-license). A very liberal commercial
license, it balances the freedoms provided by open source software with
monetization, allowing creators to earn money from their work and reinvest that
back into their efforts to continue the development long term.

You can read the reasoning behind Sustainable License in [this blog
post](https://manishrjain.com/why-sustainable-license). To better understand the license, we put together [this
FAQ](https://manishrjain.com/sustainable-license-faq).
:::

---

Outserv needs a mechanism to pay for its usage. We rely upon the Ethereum
blockchain for that. Outserv provides the `outserv wallet` tool to create a
wallet dedicated for Outserv charges. Once you create this wallet, you can then
provide it to the `outserv graphql` server for charging.

### Step 1. Create Wallet

:::tip
We highly recommend creating a wallet specifically for Outserv and keeping it
separate from your other Ethereum wallets.
:::

```bash
$ outserv wallet --create --dir ~/.wallet

Please enter password for the wallet:
Please re-enter password for the wallet:

OUTPUT:
	Created an Ethereum account with address: 0xA972C1653664CD68C9694Fd15cfE62440cdbFC1f
	Wallet JSON file stored in directory: wallet


WARNING:
	Please keep the generated JSON file and the password safe and secure.
	If you lose either of those, the funds in this account would be lost forever.

```
This would ask you for a password to use to encrypt the wallet. The wallet
would be created in outserv-wallet directory. It would be a JSON file, which
would look something like this:

```json
{
  "address": "a972c1653664cd68c9694fd15cfe62440cdbfc1f",
  "crypto": {
    "cipher": "aes-128-ctr",
    "ciphertext": "1f9ed08eb0a02a430d0505c27a66794f929dc7dade7ad129466132f061425e6a",
    "cipherparams": {
      "iv": "e64d3e56b0c656855f1182590fff6f2b"
    },
    "kdf": "scrypt",
    "kdfparams": {
      "dklen": 32,
      "n": 262144,
      "p": 1,
      "r": 8,
      "salt": "342aeb819ea4e7a0dee24adbdcf19dd009b4c8a58546f6f64c80d6147db1fddf"
    },
    "mac": "ed81d93abde4c241c24f43272d845da96a098d6656b058152189badfeaca3651"
  },
  "id": "cd1f87a6-77bc-450c-bdc1-5fd6da03455f",
  "version": 3
}
```

:::tip
Note the address of the wallet. To fund the wallet, you should send ETH to this address.
:::

### (Optional) Step 2. Test the Wallet

You can optionally test that the wallet is working by running the following
command. It would attempt to charge $1 (non-refundable). Please ensure that you
have sufficient funds to run the test.

```bash
outserv wallet --dir ~/.wallet --test
```

### Step 3. Provide Wallet to Outserv

When you run Outserv, you can pass this wallet using the `--wallet` flag.
```bash
outserv graphql --wallet="dir=~/.wallet; password=<the-password>"
```

### Step 4. Fund the Wallet

As mentioned in Step 1, the wallet has a public address where you can send ETH,
so the wallet gets funded. As you use Outserv, it would automatically charge you
for the usage, as described in the [billing
module](https://github.com/outcaste-io/outserv/tree/main/billing). **Always ensure
there are sufficient funds to pay for the usage.**

Outserv only transfers ETH between accounts. This is the cheapest operation you
can run on the Ethereum network, typically costing 21000 Wei. Outserv billing
code absorbs the gas fee charged by Ethereum, so the user doesn't have to pay
extra.

:::caution
[Sustainable License](https://manishrjain.com/sustainable-license) prohibits
tampering with the code in billing module.
:::


