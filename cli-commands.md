# Command-line programs
A list of useful CLI programs for my own memory.

## security (on MacOS)
Trust a certificate.

`security add-trusted-cert -p basic -p ssl -k <<login-keychain>> <<certificate>>'`

## dd
Convert and copy a file.  
Useful when you need to extract a part of a file into another. E.g. read the first 5mb of a large json file and dump it into a new file.

`dd if=largefile.json count=5 bs=1M > newfile.json`  

## head
Output the first part of files.  
`head -c 5mb largefile.json`  

## tail
Output the last part of a file.  
`tail -c 5mb largefile.json`  

## touch
Create a file.

## nano
Edit a file.

## chmod
Set access rights for a file or directory.