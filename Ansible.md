# Ansible Vault

## Encrypting a Variable in a yml.


 Original Yml:

```bash
cat segredo.yml
```

```yaml
---
service_user: srcuser
service_password: cleartextpassword

```

1. We'll encrypt only service_password variable
  1. Create a key_file that contains a key that will encrypt service_password variable value. Keep this file very safe and  never commit it in git (as I did here, =] )

  ```bash
    cat key_file
  ```

  ```raw
  supersecrectkey
  ```

  2. We'll use a vault-id: my_vault_id. It is used to identify uniquely the encryption payload in ansible (including ansible-playbook and awx/tower)

  3. to encrypt  the content of service_password variable insice segredo.yml, using a vault-id called my_vault_id wit a key supersecretkey located inside key_file file, run the command:

```bash
cat segredo.yml | awk -F': ' '/service_password/{printf $2}'|ansible-vault encrypt_string --encrypt-vault-id 'my_vault_id' --vault-id my_vault_id@key_file --stdin-name=service_password
```
  4. output of above command:

```
service_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;my_vault_id
          33383439643535313364393961636630383665653936393537636132653430393863613336343962
          3864313034363766623837323663393363366662383731360a663864306530393865613233653137
          33346131386334663236323431626232333335316133623737373232633835646531353266303866
          3138653964613834380a333731376239383132666162653939633034396133636431373332383231
          35393066316231306635376338393864616338353664386136383263313664343436
Encryption successful
```

  5. Create segredo_encrypted.yml with the encryped value generated above:

```yml
  ---
  service_user: srcuser
  service_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;my_vault_id
          33383439643535313364393961636630383665653936393537636132653430393863613336343962
          3864313034363766623837323663393363366662383731360a663864306530393865613233653137
          33346131386334663236323431626232333335316133623737373232633835646531353266303866
          3138653964613834380a333731376239383132666162653939633034396133636431373332383231
          35393066316231306635376338393864616338353664386136383263313664343436
  ```
  6. Now verify if you can decrypt service_password with key from key_file

  ```bash
yq -r .service_password segredo_encrypted.yml|ansible-vault --vault-password-file=key_file decrypt
  ```
  Output of above command:

  ```
  Decryption successful
  cleartextpassword
  ```
