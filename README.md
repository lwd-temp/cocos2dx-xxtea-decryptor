# cocos2dx-xxtea-decryptor
cocos2dx xxtea source decryptor (LUAC, JSC and more)

Release binary was built in x86 mode, output path of decrypted file will be located in the "parent directory+\out" folder of the file/director

Usage: exeName.exe [key] [sign] [input file/dir]

How to find xxtea key:

Search for decrypt function in the cocos2dx binary, and hook using frida, 3rd argument is normally the key


**EXAMPLE FRIDA SCRIPT TO DUMP XXTEA KEY**

The decompiled function should look similar to this, left side code is from [here](https://github.com/xpol/lua-cocos2d-x-xxtea/blob/66c3b2eb75a864baf350a191eb5a807f2028ff99/xxtea.c#L157)
![image](https://github.com/lambwheit/cocos2dx-xxtea-decryptor/assets/38606611/419dee0c-ea11-41b3-ada7-e256522ec0c1)
![image](https://github.com/lambwheit/cocos2dx-xxtea-decryptor/assets/38606611/44078423-56d0-456d-a3da-19606289868e)

```js
var module_name_libcocos2dlua_so='libcocos2dlua.so';  //change libcocos2dlua.so to the correct binary name

function start_timer_for_intercept() {
  setTimeout(
    function() {
      var offset_of_xxtea_decrypt_00a5be08=0x95be08;//update offset of decrypt function to the correct one <- can be found using ghidra
      var dynamic_address_of_xxtea_decrypt_00a5be08=Module.findBaseAddress(module_name_libcocos2dlua_so).add(offset_of_xxtea_decrypt_00a5be08);
      Interceptor.attach(dynamic_address_of_xxtea_decrypt_00a5be08, {
                 onEnter: function(args) {
                    console.log("Entered xxtea_decrypt_00a5be08");
                    console.log("[key]-> " + args[2].readCString()) // print XXTEA key 
                    //console.log('args[0]='+args[0]+' , args[1]='+args[1]+' , args[2]='+args[2]+' , args[3]='+args[3]+' , args[4]='+args[4]);
                    // this.context.x0=0x1;
                  },
                  onLeave: function(retval) {
                    //console.log("Exited xxtea_decrypt_00a5be08, retval:"+retval);
                    // retval.replace(0x1);
                  }
       });
    }, 2000);//milliseconds
}
start_timer_for_intercept();

```

How to find xxtea sign:

open multiple encrypted files in a text editor, they should have a common string at the start of the file, that is the sign
![image](https://github.com/lambwheit/cocos2dx-xxtea-decryptor/assets/38606611/3dc2fbdc-4b93-48b0-a275-59984c688b07)

In this example the sign is "akskees"
