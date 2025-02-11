# Extend Public/Private Key Expiry

These are the steps necessary to extend the expiry date for a pair of public and private keys, and where they need to be updated online.
It is written from the perspective of updating the Typelevel keys, but might (hopefully!) be useful to affiliate maintainers as well.

1. If your keys are not in `gpg`, import them:
    - `gpg --import public.asc`
    - `gpg --import private.asc`
    - run `gpg --list-keys` and ensure you see the imported key

2. `gpg --edit-key $KEY_ID` will put you into the gpg command line
    - type `expiry`, and follow the prompts to set the new expiry (we usually opt to extend the expiry by 1 year), and then type `save` to save the changes 
    - NOTE: the Typelevel key has a sub-key, and that expiry must also be extended:
      - run `gpg --edit-key $KEY_ID` again
      - type `key 1`, to edit the first sub-key
      - follow the same steps as before to update the expiry, and `save`

3. Send the updated key to the keyserver
    - `gpg --keyserver keyserver.ubuntu.com --send-keys $KEY_ID`
    - look up the key ID in the keyserver to ensure it looks correct.

4. Update the secret key in GitHub
    - `gpg --export-secret-keys $KEY_ID | base64 | pbcopy`
      - NOTE: it is important that the `base64` command being used removes newlines. This will be platform specific.
    - Update the GitHub secret variable with the copied value (For users of sbt-typelevel, this secret is named `PGP_SECRET` in GitHub)

5. Export the updated armored key files for safe keeping (e.g. in 1Password)
    - `gpg --export --armor $KEY_ID > public.asc`
    - `gpg --export-secret-keys --armor $KEY_ID > private.asc`

6. Update the public key in other locations.
    - For example, the Typelevel site has a [Web Key Directory](https://wiki.gnupg.org/WKD) that needs to be updated
      - in the Typelevel site repo, the key is at `.well-known/openpgpkey/hu/nwnwrk3rczw4ou5x56ibcrdatrgf1xag`
      - `gpg --export $KEY_ID > .well-known/openpgpkey/hu/nwnwrk3rczw4ou5x56ibcrdatrgf1xag` to update it

7. Once you are sure the private key is saved, remove the secret keys from your keyring.
    - `gpg --delete-secret-keys $KEY_ID` This will prompt you many times to be sure, and you should be sure that you have saved everything before you do this.
    - `gpg --delete-key $KEY_ID`
