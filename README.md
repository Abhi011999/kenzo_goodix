kenzo_goodix
=============

## What is it ?

These are the modified hex-dumps of goodix fingerprint blobs (and the patched blobs themselves) to work with Android 10 on kenzo.

## How did i hex-edited them ?

Well, i recently found from my friend [@magicxsavi](https://github.com/magicxavi) that a guy neamed [Arth Doshi](https://github.com/arthdoshi33) hex-edited goodix blobs for santoni device and uploaded the objdumps and hexdumps of the details
on his github [repo](https://github.com/arthdoshi33/Goodix). From there i got a clue about how and exactly what to hex-edit. So i extracted the original fingerprint dumps of kenzo and took-off.
[This](https://stackoverflow.com/a/52587878) stackoverflow answer got me almost all help about how to actually get started. I used `vscode` and `vim` on ubuntu to hex-edit them.

## What exactly did i do ?

### Step - 1 : Getting utilities -

- Run `sudo apt install vim xxd binutils-multiarch` to install all required tools.
- Use any text editor with basic find functionality (i used vscode).

### Step - 2 : Dumping instruction-set to a file -

- Run `objdump -d "blob.so" > blob.so.txt`
- You can also dump to any text format.

### Step - 3 : Dumping hex to a file -

- Run `hexdump -Cv "blob.so" > blob.so.hex`
- You can also dump to any text format.

### Step - 4 : Finding what exactly to replace -

- Now this is the crucial step, we have to find exactly the instructions we want to replace.
- For this use find utility in text editor to find a best possible match of the string or characters that you can replace.
- I found it using those blocks or chuncks of instructions seperated by name of function or class (i guess).
- Then by matching every instruction per line as you go finding for the exact instruction.
- Then when finally find it, note down its line number and hex-code somewhere (you are gonna need it).

### Step - 5 : Editing the dumped hex file -

- You will see objdump like this format - `<address of byte>:  <hex-code>  <op-code>  <instruction>`
- For example - `18684:  f85e8000  ldur  x0, [x0, #-24]`
- And hexdump like this fornat - `<address of byte>  <two hex-codes of 16 columns>  |<some ASCII chars>|`
- For example - `00018680  e1 03 18 aa 00 80 5e f8  60 00 00 8b 45 e7 ff 97  |......^.....E...|`
- Now the main part is how to find the hex-code associated with the objdump ? Well, if we search only the part of byte address excluding right-most digit in the the hexdump, we can get to the associated line easily.
- Like `18684` excluding the `4` is `1868`, when searched in hex dump, can be found at several places and can be made sure it's exactly the line we are looking for, in next steps.
- The right-most digit will consist of these four digits - `0` or `4` or `8` or `c`, and that's how you will determine which octet (hex-code) to look.
- `0` means first octet, example by above dump - `e1 03 18 aa`,
- `c` means fourth octet, example - `45 e7 ff 97`, etc.
- After we have found the line and the octet, we will move onto making sure we have found right line. If you look at the obdump's hex-code in reverse by two characters taken together, you can see that they are similar to the hex-codes in the hex-dump.
- Like `f85e8000` in reverse with two characters taken together is `00 80 5e f8`, which is in second octet as the last digit `4` of the byte address in objdump mentions it. Hence Verified !
- Now we will replace the octet with `no-op`'s hex-code, which is `1f 20 03 d5`, equivalent to `d503201f` in objdump's hex-code.
- Replace all the required octets with above `1f 20 03 d5` by following above steps and we are done hex-editing !

### Step - 6 : Applying edited changes in original blob -

- Run `xxd -g 1 <blob.so> | vim -`
- Above command will open the original blob in hex mode, from there we need to find the line of hex-code.
- Typing `/` will bring the find command in vim and you can search for the byte address using step-5.
- After finding the appropriate octet you have to then enter the vim's insert mode.
- Typing `:i` will get you to insert mode and from there you can copy paste the octet from hex-dump file.
- Press `Esc` when done making changes and vim will exit from insert mode.
- Repeat above steps to replace all octets required.
- When done making changes then type `:%!xxd -r > new_blob.so`
- Above command will revert the patched hex-dump in vim editor back to original blob and create a `new_blob.so` file.
- We have done it ! You can see the `new_blob.so` in your working directory.
- Type `:q!` to exit vim.

### Step - 7 : Making sure new blob is correctly patched -

- Run `objdump -d new_blob.so > new_blob.so.txt`
- If everything is patched correctly then it will generate the obj-dump with no errors.
- If it generated errors or un-identifies file-type then you did something wrong.
- You can then open the new blob's obj-dump and go to the line which you chose to edit and see that they are replaced by `18684:  d503201f  nop`

**All Done ! Use your new blob !**

## What and who helped me ?

- First and foremost this repo - https://github.com/arthdoshi33/Goodix hinted me exactly what to edit.
- This link explains hex-editing is good detail - https://stackoverflow.com/a/52587878
- Vim editor's help - https://askubuntu.com/questions/256782/how-to-copy-paste-contents-in-the-vi-editor
- Patching with vim - https://vi.stackexchange.com/questions/343/how-to-edit-binary-files-with-vim
- ARM64 Instruction Set manual (if you are curious) - https://static.docs.arm.com/100898/0100/the_a64_Instruction_set_100898_0100.pdf

## Special Thanks

- [@magicxsavi](https://github.com/magicxavi) for pointing out the [Arth's](https://github.com/arthdoshi33) [repo](https://github.com/arthdoshi33/Goodix)
- Cheer's to all my testers as i don't have the goodix device personally, i rely on them for feedback and logs.