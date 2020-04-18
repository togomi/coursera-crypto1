# **Maryland Programming Assignment (Week 1)**
Below are 7 ciphertexts, each of which was generated by encrypting some 31-character ASCII plaintext with the one-time pad using the same key (code for the encryption program used is given below).

Decrypt them and recover all 7 plaintexts, each of which is a grammatically correct English sentence.

Note: for this problem it is easiest to use a combination of automated analysis plus human insight and even occasional guessing. As long as you can decrypt them all, it doesn't matter how you do it.
```
BB3A65F6F0034FA957F6A767699CE7FABA855AFB4F2B520AEAD612944A801E
BA7F24F2A35357A05CB8A16762C5A6AAAC924AE6447F0608A3D11388569A1E
A67261BBB30651BA5CF6BA297ED0E7B4E9894AA95E300247F0C0028F409A1E
A57261F5F0004BA74CF4AA2979D9A6B7AC854DA95E305203EC8515954C9D0F
BB3A70F3B91D48E84DF0AB702ECFEEB5BC8C5DA94C301E0BECD241954C831E
A6726DE8F01A50E849EDBC6C7C9CF2B2A88E19FD423E0647ECCB04DD4C9D1E
BC7570BBBF1D46E85AF9AA6C7A9CEFA9E9825CFD5E3A0047F7CD009305A71E
```

They were generated using the following program:
```C
#include <stdio.h>
#define KEY_LENGTH 31

int main()
{
    unsigned char ch;
    FILE *fpIn, *fpOut;
    int i;
    unsigned char key[KEY_LENGTH] = { 0x00, 0x00, 0x00, 0x00,
                                        0x00, 0x00, 0x00, 0x00,
                                        0x00, 0x00, 0x00, 0x00,
                                        0x00, 0x00, 0x00, 0x00,
                                        0x00, 0x00, 0x00, 0x00,
                                        0x00, 0x00, 0x00, 0x00,
                                        0x00, 0x00, 0x00, 0x00,
                                        0x00, 0x00, 0x00};
    // Of course, I did not use the all-0 key when generating the 7 ciphertexts above!

    fpIn = fopen("messages.txt", "r");
    fpOut = fopen("ctexts.txt", "w");

    i=0;

    while (fscanf(fpIn, "%c", &ch) != EOF)
    {
        fprintf(fpOut, "%02X", ch^key[i]);
        i++;
        if (i==31)
        {
            fprintf(fpOut, "\n");
            i=0;
            fscanf(fpIn, "%c", &ch);
        }
    }

    fclose(fpIn);
    fclose(fpOut);

    return;
}
```


## **Approach 1**

Tried to re-use VengenereCipher code...

This is not bad.. but it is still critical to determine the space character.

**Output**
```
$ gcc -std=c99 decrypt_vengenere.c -o decrypt_vengenere.exe && ./decrypt_vengenere.exe

d am ppahntng a sncryt missiowae
e is hhc rnly peysor to trusmay
he cunrcni plan bs hop secremaz
hen stosly we ment ho do thijpd
 thinw rhxy shougd zollow hitay
his io vuoer thae ttat one ijac
ot ony eayet is iether than Pa

```

## **Approach 2**
Given PTs only consist of alphabet and space and many OTP-encrypted CTs with the same key, we can figure out existence of "space" by XORing CTs.

**Output**

```
$ g++ -std=c++11 decrypt_otp.cpp -o decrypt_otp.exe && ./decrypt_otp.exe
---------------------------------------------------------------
Each number represents number of times it XOR to produce space
---------------------------------------------------------------
0512410301021426201301531111100
0262360306021351201306236111100
0215310301051321501401241111100
0212410301051351201401531611100
0512310401026321201401231161100
0212410401021421206301241116100
0215310401021421501301241111600
---------------------------------------------------------------
0012010401024010205201021345600  <- index having the highest count
---------------------------------------------------------------
* am p*a*n*ng a s*cr*t missio**
*e is *h* *nly pe*so* to trus**
*he cu*r*n* plan *s *op secre**
*hen s*o*l* we me*t *o do thi**
* thin* *h*y shou*d *ollow hi**
*his i* *u*er tha* t*at one i**
*ot on* *a*et is *et*er than **
```

This looks pretty good. If space is not found, we cannot find the key - shown as "*".

We can guess some unknown ("*") characters in the decrypted PT sentences and make following gues:
```c++
keys[0] = 'T' ^ ciphers[5][0];
keys[6] = 'k' ^ ciphers[4][6];
keys[8] = 'e' ^ ciphers[2][8];
keys[10] = 'i' ^ ciphers[0][10];
keys[17] = 'e' ^ ciphers[0][17];
keys[20] = 'e' ^ ciphers[0][20];
keys[29] = 'n' ^ ciphers[0][29];
keys[30] = '.' ^ ciphers[0][30];
```

```
---------------------------------------------------------------
Guessed rest of the keys from roughly decrypted text
---------------------------------------------------------------
I am planning a secret mission.
He is the only person to trust.
The current plan is top secret.
When should we meet to do this?
I think they should follow him.
This is purer than that one is.
Not one cadet is better than I.
```

---
---

# **Stanford Programming Assignment (Week 1)**
Many Time Pad

Let us see what goes wrong when a stream cipher key is used more than once. Below are eleven hex-encoded ciphertexts that are the result of encrypting eleven plaintexts with a stream cipher, all with the same stream cipher key. Your goal is to decrypt the last ciphertext, and submit the secret message within it as solution.

Hint: XOR the ciphertexts together, and consider what happens when a space is XORed with a character in [a-zA-Z].

```
315c4eeaa8b5f8aaf9174145bf43e1784b8fa00dc71d885a804e5ee9fa40b16349c146fb778cdf2d3aff021dfff5b403b510d0d0455468aeb98622b137dae857553ccd8883a7bc37520e06e515d22c954eba5025b8cc57ee59418ce7dc6bc41556bdb36bbca3e8774301fbcaa3b83b220809560987815f65286764703de0f3d524400a19b159610b11ef3e
234c02ecbbfbafa3ed18510abd11fa724fcda2018a1a8342cf064bbde548b12b07df44ba7191d9606ef4081ffde5ad46a5069d9f7f543bedb9c861bf29c7e205132eda9382b0bc2c5c4b45f919cf3a9f1cb74151f6d551f4480c82b2cb24cc5b028aa76eb7b4ab24171ab3cdadb8356f
32510ba9a7b2bba9b8005d43a304b5714cc0bb0c8a34884dd91304b8ad40b62b07df44ba6e9d8a2368e51d04e0e7b207b70b9b8261112bacb6c866a232dfe257527dc29398f5f3251a0d47e503c66e935de81230b59b7afb5f41afa8d661cb
32510ba9aab2a8a4fd06414fb517b5605cc0aa0dc91a8908c2064ba8ad5ea06a029056f47a8ad3306ef5021eafe1ac01a81197847a5c68a1b78769a37bc8f4575432c198ccb4ef63590256e305cd3a9544ee4160ead45aef520489e7da7d835402bca670bda8eb775200b8dabbba246b130f040d8ec6447e2c767f3d30ed81ea2e4c1404e1315a1010e7229be6636aaa
3f561ba9adb4b6ebec54424ba317b564418fac0dd35f8c08d31a1fe9e24fe56808c213f17c81d9607cee021dafe1e001b21ade877a5e68bea88d61b93ac5ee0d562e8e9582f5ef375f0a4ae20ed86e935de81230b59b73fb4302cd95d770c65b40aaa065f2a5e33a5a0bb5dcaba43722130f042f8ec85b7c2070
32510bfbacfbb9befd54415da243e1695ecabd58c519cd4bd2061bbde24eb76a19d84aba34d8de287be84d07e7e9a30ee714979c7e1123a8bd9822a33ecaf512472e8e8f8db3f9635c1949e640c621854eba0d79eccf52ff111284b4cc61d11902aebc66f2b2e436434eacc0aba938220b084800c2ca4e693522643573b2c4ce35050b0cf774201f0fe52ac9f26d71b6cf61a711cc229f77ace7aa88a2f19983122b11be87a59c355d25f8e4
32510bfbacfbb9befd54415da243e1695ecabd58c519cd4bd90f1fa6ea5ba47b01c909ba7696cf606ef40c04afe1ac0aa8148dd066592ded9f8774b529c7ea125d298e8883f5e9305f4b44f915cb2bd05af51373fd9b4af511039fa2d96f83414aaaf261bda2e97b170fb5cce2a53e675c154c0d9681596934777e2275b381ce2e40582afe67650b13e72287ff2270abcf73bb028932836fbdecfecee0a3b894473c1bbeb6b4913a536ce4f9b13f1efff71ea313c8661dd9a4ce
315c4eeaa8b5f8bffd11155ea506b56041c6a00c8a08854dd21a4bbde54ce56801d943ba708b8a3574f40c00fff9e00fa1439fd0654327a3bfc860b92f89ee04132ecb9298f5fd2d5e4b45e40ecc3b9d59e9417df7c95bba410e9aa2ca24c5474da2f276baa3ac325918b2daada43d6712150441c2e04f6565517f317da9d3
271946f9bbb2aeadec111841a81abc300ecaa01bd8069d5cc91005e9fe4aad6e04d513e96d99de2569bc5e50eeeca709b50a8a987f4264edb6896fb537d0a716132ddc938fb0f836480e06ed0fcd6e9759f40462f9cf57f4564186a2c1778f1543efa270bda5e933421cbe88a4a52222190f471e9bd15f652b653b7071aec59a2705081ffe72651d08f822c9ed6d76e48b63ab15d0208573a7eef027
466d06ece998b7a2fb1d464fed2ced7641ddaa3cc31c9941cf110abbf409ed39598005b3399ccfafb61d0315fca0a314be138a9f32503bedac8067f03adbf3575c3b8edc9ba7f537530541ab0f9f3cd04ff50d66f1d559ba520e89a2cb2a83
32510ba9babebbbefd001547a810e67149caee11d945cd7fc81a05e9f85aac650e9052ba6a8cd8257bf14d13e6f0a803b54fde9e77472dbff89d71b57bddef121336cb85ccb8f3315f4b52e301d16e9f52f904
```

Target ciphertext to decrypt (last in above)
```
32510ba9babebbbefd001547a810e67149caee11d945cd7fc81a05e9f85aac650e9052ba6a8cd8257bf14d13e6f0a803b54fde9e77472dbff89d71b57bddef121336cb85ccb8f3315f4b52e301d16e9f52f904
```

For completeness, here is the python script used to generate the ciphertexts.

(it doesn't matter if you can't read this)
```python
import sys

MSGS = ( ---  11 secret messages  --- )

def strxor(a, b):     # xor two strings of different lengths
    if len(a) > len(b):
       return "".join([chr(ord(x) ^ ord(y)) for (x, y) in zip(a[:len(b)], b)])
    else:
       return "".join([chr(ord(x) ^ ord(y)) for (x, y) in zip(a, b[:len(a)])])

def random(size=16):
    return open("/dev/urandom").read(size)

def encrypt(key, msg):
    c = strxor(key, msg)
    print
    print c.encode('hex')
    return c

def main():
    key = random(1024)
    ciphertexts = [encrypt(key, msg) for msg in MSGS]
```

**Output**
```
g++ -std=c++11 decrypt_otp_stanford.cpp -o decrypt_otp_stanford.exe && ./decrypt_otp_stanford.exe
---------------------------------------------------------------
First, Lets see the guessed key
---------------------------------------------------------------
66.39.6E.89.C9.DB.D8.CB.98.74.35.2A.CD.63.95.10.2E.AF.CE.78.AA.7F.ED.28.A0.6E.7E.C9.8D.29.C5.0B.69.B0.33.DB.14.F8.AA.8F.1A.9C.6D.70.8F.80.C0.66.C7.63.F0.F0.12.31.48.CD.D8.E8.02.D0.5B.A9.87.77.33.5D.AE.FC.EC.D5.9C.43.3A.6B.26.8B.60.BF.4E.F0.3C.9A.70.05.20.20.20.20.31.61.ED.20.20.04.20.7B.76.CF.D2.20.D2.20.8C.57.37.6E.DB.A8.C2.20.FFFFFFFF.4F.7C.FFFFFFFF.76.61.E2.A1.20.20.45.20.44.50.53.C0.A1.BA.FFFFFFFF.6C.78.FFFFFFFF.91.79.41.FFFFFFFF.FFFFFFFF.CF.FFFFFFFF.BB.C6.43.4A.C4.AB.41.87.FFFFFFFF.A9.FFFFFFFF.BF.57.8C.C7.8A.A8.82.D1.B9.A3.67.FFFFFFFF.FFFFFFFF.9E.A7.85.BC.FFFFFFFF.7D.4C.D8.C4.91.FFFFFFFF.FFFFFFFF.DF.D7.FFFFFFFF.83.FFFFFFFF.E8.46.FFFFFFFF.F9.84.EE.
---------------------------------------------------------------

---------------------------------------------------------------
Lets see the decryped message using guessed key
---------------------------------------------------------------
[00]: We can aactor the number    with qu ctu▒ computers  We can also factor the number   ▒▒w▒h a▒▒o▒n raKn▒d to ba▒*mt* he EmG  n Ro*,r*   ** *
[01]: Euler whuld probably enjoh5that nowaeis▒theorem bemomes a corner stone of crypto -1T▒▒q▒ymo▒▒ ▒ tEuNe▒'s theo▒*
[02]: The nicb thing about Keey}zq is nowaze ▒ryptographkrs can drive a lot of fancy carb5▒▒Z▒n B▒▒e▒
[03]: The cipoertext produced bh5a weak e/nry▒tion algorgthm looks as good as ciphertext1e▒▒z▒ced▒▒y▒/tstPo▒g encry▒*$o*rllgd^iV;mc- P* l*pH
[04]: You don t want to buy a sta of car *hys▒from a guy.who specializes in stealing carb5▒▒S▒rc ▒▒t▒ 6erE ▒ommenti▒*mo*rNli{\eP
a****r4.;rd  *e*          u**    * i  phya  t▒at which wgll keep secrets safe from your }|▒▒r▒ si▒▒e▒btanF ▒hat whi▒*mw*>a knIp e ret*is*f
[06]: There aue two types of cyaaography:abne▒that allow} the Government to use brute focvݻj▒ br▒▒k▒:<e Ao▒e, and ▒*( *:lt yIqW:r&s t*, *o$**(*<9a:od2<* *<81+tfbr7 ** 1-*. <= **  * *  *
[07]: We can tee the point whert5the chipads ▒nhappy if o wrong bit is sent and consumes1x▒▒{▒pow▒▒ ▒<;m Vh▒ enviro▒*(n*r  AoE q;a.ir
[08]: A (privfte-key)  encrypti~{ scheme 2yat▒s 3 algorizhms, namely a procedure for gentg▒▒w▒g k▒▒s▒n5 pPo▒edure f▒*me*1ypEnE "nd *ip*o
                                                                                                                                        $**7*r+.<  ",*y*:$+)z▒
[09]:  The Coicise OxfordDictiotry (2006h-de ▒▒nes crypzo as the art of  writing o r so}c▒▒y▒cod▒▒.▒
[10]: The secuet message is: Wht{ using aa~tr▒am cipher,.never use the key more than onct
---------------------------------------------------------------

---------------------------------------------------------------
Unset key if the decrypted text is not readable
---------------------------------------------------------------
66.39.6E.89.C9.DB.D8.CB.98.74.35.2A.CD.63.95.10.2E.AF.CE.78.AA.7F.ED.28.A0.FFFFFFFF.7E.C9.8D.29.C5.0B.69.B0.33.DB.14.F8.AA.FFFFFFFF.FFFFFFFF.FFFFFFFF.6D.70.8F.80.C0.66.C7.63.F0.F0.12.31.48.CD.D8.E8.02.D0.5B.A9.87.77.33.5D.AE.FC.EC.D5.9C.43.3A.6B.26.8B.60.BF.4E.F0.3C.9A.70.FFFFFFFF.FFFFFFFF.FFFFFFFF.20.FFFFFFFF.31.61.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.7B.76.FFFFFFFF.FFFFFFFF.20.FFFFFFFF.FFFFFFFF.FFFFFFFF.57.37.6E.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.4F.7C.FFFFFFFF.76.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.20.45.FFFFFFFF.FFFFFFFF.50.53.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.6C.78.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.43.4A.FFFFFFFF.FFFFFFFF.41.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.57.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.67.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.FFFFFFFF.BC.FFFFFFFF.7D.4C.D8.C4.91.FFFFFFFF.FFFFFFFF.DF.D7.FFFFFFFF.83.FFFFFFFF.E8.46.FFFFFFFF.F9.84.EE.

---------------------------------------------------------------
Removed unreadable part
---------------------------------------------------------------
[00]: We can aactor the number *  with qu ctu***omputers  We can also factor the number  ***w*h *****n **K*** to*****mt* ****Em** n****,r********
[01]: Euler whuld probably enjo*5that nowaeis***eorem bemomes a corner stone of crypto -1***q*ym***** t**N***s t*****
[02]: The nicb thing about Keey*zq is nowaze ***ptographkrs can drive a lot of fancy carb***Z*n *****
[03]: The cipoertext produced b*5a weak e/nry***on algorgthm looks as good as ciphertext1***z*ce*****/t**P*** en*****$o*r****^i**mc**** l**********  *
[04]: You don t want to buy a s*a of car *hys***om a guy.who specializes in stealing carb***S*rc***** 6**E***mme*****mo*r****\e*
[05]: There aue two types of cr*etographya  t*** which wgll keep secrets safe from your }***r* s*****bt**F***at *****mw*>****Ip**e ****is**********.;** ***** ********u***** * i
[06]: There aue two types of cy*aography:abne***at allow} the Government to use brute foc***j* b*****:<**A***, a*****( *:****Iq**r&****, **********a:**2*****8******** *****-*. <= **  * *  *
[07]: We can tee the point wher*5the chipads ***appy if o wrong bit is sent and consumes1***{*po*****<;**V***env*****(n*r****E **a.**
[08]: A (privfte-key)  encrypti*{ scheme 2yat***3 algorizhms, namely a procedure for gent***w*g *****n5**P***dur*****me*1****En** "****ip**********.<**"*****$****
[09]:  The Coicise OxfordDictio*try (2006h-de***nes crypzo as the art of  writing o r so}***y*co*****
[10]: The secuet message is: Wh*{ using aa~tr*** cipher,.never use the key more than onct
---------------------------------------------------------------

---------------------------------------------------------------
Corrected Keys...
---------------------------------------------------------------
66.39.6E.89.C9.DB.D8.CC.98.74.35.2A.CD.63.95.10.2E.AF.CE.78.AA.7F.ED.28.A0.7F.6B.C9.8D.29.C5.0B.69.B0.33.9A.19.F8.AA.40.1A.9C.6D.70.8F.80.C0.66.C7.63.FE.F0.12.31.48.CD.D8.E8.02.D0.5B.A9.87.77.33.5D.AE.FC.EC.D5.9C.43.3A.6B.26.8B.60.BF.4E.F0.3C.9A.61.10.98.BB.3E.9A.31.61.ED.C7.B8.04.A3.35.22.CF.D2.02.D2.C6.8C.57.37.6E.DB.A8.C2.CA.50.02.7C.61.24.6C.E2.A1.2B.0C.45.02.17.50.10.C0.A1.BA.46.25.78.6D.91.11.00.79.7D.8A.47.E9.8B.02.04.C4.EF.06.C8.67.A9.50.F1.1A.C9.89.DE.A8.8F.D1.DB.F1.67.48.74.9E.D4.C6.F4.5B.38.4C.9D.96.C4.FFFFFFFF.FFFFFFFF.DF.D7.FFFFFFFF.83.FFFFFFFF.E8.46.FFFFFFFF.F9.84.EE.
---------------------------------------------------------------

[00]: We can factor the number 15 with quantum computers. We can also factor the number 15 with a dog trained to bark three times - Robert Harley
[01]: Euler would probably enjoy that now his theorem becomes a corner stone of crypto - Annonymous on Euler's theorem
[02]: The nice thing about Keeyloq is now we cryptographers can drive a lot of fancy cars - Dan Boneh
[03]: The ciphertext produced by a weak encryption algorithm looks as good as ciphertext produced by a strong encryption algorithm - Philip Zimmermann
[04]: You don't want to buy a set of car keys from a guy who specializes in stealing cars - Marc Rotenberg commenting on Clipper
[05]: There are two types of cryptography - that which will keep secrets safe from your little sister, and that which will keep secrets safe from your government - Bruce Schneier
[06]: There are two types of cyptography: one that allows the Government to use brute force to break the code, and one that requires the Government to use brute force to break you**  * *  *
[07]: We can see the point where the chip is unhappy if a wrong bit is sent and consumes more power from the environment - Adi Shamir
[08]: A (private-key)  encryption scheme states 3 algorithms, namely a procedure for generating keys, a procedure for encrypting, and a procedure for decrypting.▒
[09]:  The Concise OxfordDictionary (2006) deﬁnes crypto as the art of  writing o r solving codes.
[10]: The secret message is: When using a stream cipher, never use the key more than once
---------------------------------------------------------------
```