KMS 
- Any time you hear "encryption" it's probably KMS - EBS volumes, S3, redshift, RDS, SSM Parameter Store, etc
- Key store, have some control over it but not all
- Easy way to control access for your data
- Can also use CLI to encrypt data yourself
- Customer Master Key (CMK) never can be retrieved by the user
  - Can be rotated for extra security
- Never store secrets in plaintext - Could encrypt secrets, store in code/env vars
- Only up to 4KB data per call - creds, certificates, etc.
  - If > 4KB, use envelope encryption
- To give access
  1) key policy allows access to key - on a per key basis; default or you can set specific set of key administrators and key usage permissions; default key policy allows root user to do anything. Also can allow use of IAM permissions, which is how users get access to use the key.
  2) IAM policy allows access
- Fully manage keys - create, rotate, disable, audit (CloudTrail)
- Three types of CMK
  - AWS default CMK: free
  - User keys create in KMS: $1/month
  - User keys imported (256 bit symmetric key): $1/month
- API - Encrypt and Decrypt
  - Example: want to encrypt a secret
    - Send to Encrypt Api
	- KMS checks IAM permissions
	- KMS encrypts using CMK
	- Returns encrypted secrets

KMS Encryption SDK - need to know when to use it
- What if you want to encrypt over 4KB? E.g. EBS volumes
- Need to use Envelope Encryption! Use the SDK
- NOTE: Different from S3 encryption SDK
- Also has a CLI tool
- EXAM: Anything over 4KB of data - (1) Use Encryption SDK (2) Envelope Encryption (3) Use GenerateDataKey API and client-side encryption
- Client big file encryption - same as above except use GenerateDataKey API
- GenerateDataKey API
  - KMS creates key and encrypts it
  - KMS returns plaintext key and encrypted key to client
  - client use the Plaintext data encryption key to encrypt the file
  - create "Envelope" consisting of encrypted file + encrypted data key
  - store the Envelope wherever (e.g. S3)
- Use the simple Decrypt API to decrypt the key and decrypt the envelope  
- SDK does all this for us automatically
  aws-encryption-cli --encrypt -> encrypted envelope file
