---
title: "Cyber Apocalypse 2023"
date: 2023-03-25 08:00:00 +0800
categories:
  - "CTF"
  - "Cyber Apocalypse 2023"
tags: [crypto]
image:
    path: /assets/img/cyber_apocalyse_2023/cyber_apocalypse_2023_banner.png
---

Decided to try some `very easy` difficulty cryptography challenges on Cyber Apocalyse 2023 organised by Hack The Box (HTB).

## Ancient Encodings

> Your initialization sequence requires loading various programs to gain the necessary knowledge and skills for your journey. Your first task is to learn the ancient encodings used by the aliens in their communication.
> | Downloadable File | SHA256 checksum |
> | --- | --- |
> | [crypto_ancient_encodings.zip](/assets/img/cyber_apocalypse_2023/crypto_ancient_encodings.zip) | 9b45c2d7a3b3835ca4966ebcf4b8a999e46cf2dc666e5b4dc919bd92d5996379 |

### Solution

Observed two files, `output.txt` and `source.py`.

`Output.txt` contains a hex string to be decoded.

```
0x53465243657a467558336b7764584a66616a4231636d347a655639354d48566664326b786246397a5a544e66644767784e56396c626d4d775a4446755a334e665a58597a636e6c33614756794d33303d
```

Observed that the flag has been encoded in `source.py`.

```python
def encode(message):
    return hex(bytes_to_long(b64encode(message)))
```

Prepare `solution.py`.

```python
from base64 import b64decode

with open('output.txt','r') as f:
    encoded_flag = f.read()

decoded_flag = b64decode(bytes.fromhex(encoded_flag.strip('0x')))

print(decoded_flag)
```

Obtained flag.

```bash
b'HTB{1n_y0ur_j0urn3y_y0u_wi1l_se3_th15_enc0d1ngs_ev3rywher3}'
```

### Flag

`HTB{1n_y0ur_j0urn3y_y0u_wi1l_se3_th15_enc0d1ngs_ev3rywher3}`

## Small StEps

> As you continue your journey, you must learn about the encryption method the aliens used to secure their communication from eavesdroppers. The engineering team has designed a challenge that emulates the exact parameters of the aliens' encryption system, complete with instructions and a code snippet to connect to a mock alien server. Your task is to break it.
> | Downloadable File | SHA256 checksum |
> | --- | --- |
> | [crypto_small_steps.zip](/assets/img/cyber_apocalypse_2023/crypto_small_steps.zip) | 81cff3a0df3eec1137ee73b8885fc186f68f9b92b62227840f9187619a82e39c |

### Solution

Observed from `server.py` that this is a RSA challenge.

```python
class RSA:

    def __init__(self):
        self.q = getPrime(256)
        self.p = getPrime(256)
        self.n = self.q * self.p
        self.e = 3

    def encrypt(self, plaintext):
        plaintext = bytes_to_long(plaintext)
        return pow(plaintext, self.e, self.n)
```

Interacted with the docker spawned.

![image](/assets/img/cyber_apocalypse_2023/2795be400ad8c78e7c69165fac95b6a2c8f2a02d5866c0e84b8094e55dbed6bf.png)  

Summarising the important variables.

```
N:
5375983221960016614973592857478881893323596529918316933926644498645437217175230065318969389265697863653504543601699830880628339321663999903790995971224041
e:
3
encrypted flag:
70407336670535933819674104208890254240063781538460394662998902860952366439176467447947737680952277637330523818962104685553250402512989897886053
```

Used RsaCtfTool to solve. Reference: <https://github.com/RsaCtfTool/RsaCtfTool>

```python
$ python RsaCtfTool.py -n 7123985100617323664645174986225121094324414856672926376473228164015644277173979816221519172436493463188178653211572222750286328836858766812354896656252193 -e 3 --uncipher 70407336670535933819674104208890254240063781538460394662998902860952366439176467447947737680952277637330523818962104685553250402512989897886053
```

![image](/assets/img/cyber_apocalypse_2023/2e10217c5ae116015b87c0e31934db7a2ed3ea60a4e2f0f8c0fc4b67e3cb9c33.png)  

### Flag

`HTB{5ma1l_E-xp0n3nt}`

## Perfect Synchronization

> The final stage of your initialization sequence is mastering cutting-edge technology tools that can be life-changing. One of these tools is quipqiup, an automated tool for frequency analysis and breaking substitution ciphers. This is the ultimate challenge, simulating the use of AES encryption to protect a message. Can you break it?
> | Downloadable File | SHA256 checksum |
> | --- | --- |
> | [crypto_perfect_synchronization.zip](/assets/img/cyber_apocalypse_2023/crypto_perfect_synchronization.zip) | fbd1224b93d9ec01d5e6464f70b11e58f396ae73803a4d563d35f498d524ef67 |

### Solution

Observed two files, `output.txt` and `source.py`.

`output.txt` contains similar length hex values separated by newlines.

```
dfc8a2232dc2487a5455bda9fa2d45a1
305d4649e3cb097fb094f8f45abbf0dc
c87a7eb9283e59571ad0cb0c89a74379
60e8373bfb2124aea832f87809fca596
d178fac67ec4e9d2724fed6c7b50cd26
c87a7eb9283e59571ad0cb0c89a74379
34ece5ff054feccc5dabe9ae90438f9d
457165130940ceac01160ac0ff924d86
5d7185a6823ab4fc73f3ea33669a7bae
61331054d82aeec9a20416759766d9d5
-----Redacted-----
```

Observed in `source.py` that AES ECB is implemented.

```python
class Cipher:
    def __init__(self):
        self.salt = urandom(15)
        key = urandom(16)
        self.cipher = AES.new(key, AES.MODE_ECB)

    def encrypt(self, message):
        return [self.cipher.encrypt(c.encode() + self.salt) for c in message]

def main():
    cipher = Cipher()
    encrypted = cipher.encrypt(MESSAGE)
    encrypted = "\n".join([c.hex() for c in encrypted])

    with open("output.txt", 'w+') as f:
        f.write(encrypted)
```

Use a python script to perform the following actions.

```python
with open("output.txt") as f:
    word_list = f.readlines()

# Remove new line characters
word_list = [x.strip() for x in word_list]

# Create a set of unique words from the list and an empty dictionary
unique_word = set(word_list)
word_dict = {}

# Assign a unique character to each unique word
for index, word in enumerate(unique_word):
    word_dict[word] = chr(65 + index)

# Combine to form the original ciphertext
test_text = ''.join([word_dict[word] for word in word_list])

print(test_text)
```

Obtained the following output for `test_text`.

```
XJHETHNYPVLNLGPWDWVDWVMLWH]VSNVZQHVXLYZVZQLZVDNVLNPVODBHNVWZJHZYQVSXVIJDZZHNVGLNOTLOHVYHJZLDNVGHZZHJWVLN]VYSUMDNLZDSNWVSXVGHZZHJWVSYYTJVIDZQVBLJPDNOVXJHETHNYDHWVUSJHSBHJVZQHJHVDWVLVYQLJLYZHJDWZDYV]DWZJDMTZDSNVSXVGHZZHJWVZQLZVDWVJSTOQGPVZQHVWLUHVXSJVLGUSWZVLGGVWLU[GHWVSXVZQLZVGLNOTLOHVDNVYJP[ZLNLGPWDWVXJHETHNYPVLNLGPWDWVLGWSVKNSINVLWVYSTNZDNOVGHZZHJWVDWVZQHVWZT]PVSXVZQHVXJHETHNYPVSXVGHZZHJWVSJVOJST[WVSXVGHZZHJWVDNVLVYD[QHJZHAZVZQHVUHZQS]VDWVTWH]VLWVLNVLD]VZSVMJHLKDNOVYGLWWDYLGVYD[QHJWVXJHETHNYPVLNLGPWDWVJHETDJHWVSNGPVLVMLWDYVTN]HJWZLN]DNOVSXVZQHVWZLZDWZDYWVSXVZQHV[GLDNZHAZVGLNOTLOHVLN]VWSUHV[JSMGHUVWSGBDNOVWKDGGWVLN]VDXV[HJXSJUH]VMPVQLN]VZSGHJLNYHVXSJVHAZHNWDBHVGHZZHJVMSSKKHH[DNOV]TJDNOVISJG]VILJVDDVMSZQVZQHVMJDZDWQVLN]VZQHVLUHJDYLNWVJHYJTDZH]VYS]HMJHLKHJWVMPV[GLYDNOVYJSWWISJ]V[TFFGHWVDNVULCSJVNHIW[L[HJWVLN]VJTNNDNOVYSNZHWZWVXSJVIQSVYSTG]VWSGBHVZQHUVZQHVXLWZHWZVWHBHJLGVSXVZQHVYD[QHJWVTWH]VMPVZQHVLADWV[SIHJWVIHJHVMJHLKLMGHVTWDNOVXJHETHNYPVLNLGPWDWVXSJVHALU[GHVWSUHVSXVZQHVYSNWTGLJVYD[QHJWVTWH]VMPVZQHVCL[LNHWHVUHYQLNDYLGVUHZQS]WVSXVGHZZHJVYSTNZDNOVLN]VWZLZDWZDYLGVLNLGPWDWVOHNHJLGGPVQZM\LRWDU[GHRWTMWZDZTZDSNRDWRIHLK^VYLJ]VZP[HVULYQDNHJPVIHJHVXDJWZVTWH]VDNVISJG]VILJVDDV[SWWDMGPVMPVZQHVTWVLJUPWVWDWVZS]LPVZQHVQLJ]VISJKVSXVGHZZHJVYSTNZDNOVLN]VLNLGPWDWVQLWVMHHNVJH[GLYH]VMPVYSU[TZHJVWSXZILJHVIQDYQVYLNVYLJJPVSTZVWTYQVLNLGPWDWVDNVWHYSN]WVIDZQVUS]HJNVYSU[TZDNOV[SIHJVYGLWWDYLGVYD[QHJWVLJHVTNGDKHGPVZSV[JSBD]HVLNPVJHLGV[JSZHYZDSNVXSJVYSNXD]HNZDLGV]LZLV[TFFGHV[TFFGHV[TFFGH
```

Use quipqiup to solve the test_text. Reference: <https://quipqiup.com/>

![image](/assets/img/cyber_apocalypse_2023/98d0b5dcf14c1f26a286d8aa74d7948439af98f1ffc4d2b3609cbb39494f64fa.png)  


Searching on Google, this seemed to be text from a Wikipedia article on Frequency Analysis. Reference: <https://en.wikipedia.org/wiki/Frequency_analysis>

![image](/assets/img/cyber_apocalypse_2023/036d6a8ebc43695bb9f046b0eb47b3f698064f9db8849c143ffa631f61b02a4d.png)  

Since this is a substitution cipher, implement a python script to guess the characters.

```python
import string

with open("output.txt") as f:
    word_list = f.readlines()

# remove new line characters
word_list = [x.strip() for x in word_list]

# Create a set of unique items from the list and an empty dictionary
unique_word = set(word_list)
word_dict = {}

# Use lowercase to indicate unsolved characters
char_list = list(string.ascii_lowercase + string.octdigits)

# Assign a unique character to each unique item
for index, word in enumerate(unique_word):
    word_dict[word] = char_list[index]

word_dict |= {'dfc8a2232dc2487a5455bda9fa2d45a1':'F',
              '305d4649e3cb097fb094f8f45abbf0dc':'R',
              'c87a7eb9283e59571ad0cb0c89a74379':'E',
              '60e8373bfb2124aea832f87809fca596':'Q',
              'd178fac67ec4e9d2724fed6c7b50cd26':'U',
              '34ece5ff054feccc5dabe9ae90438f9d':'N',
              '457165130940ceac01160ac0ff924d86':'C',
              '5d7185a6823ab4fc73f3ea33669a7bae':'Y',
              '61331054d82aeec9a20416759766d9d5':' ',
              '5f122076e17398b7e21d1762a61e2e0a':'A',
              'f89f2719fb2814d9ab821316dae9862f':'L',
              '200ecd2657df0197f202f258b45038d8':'S',
              'e9b131ab270c54bbf67fb4bd9c8e3177':'I',
              '9673dbe632859fa33b8a79d6a3e3fe30':'B',
              '200ecd2657df0197f202f258b45038d8':'S',
              'e23c1323abc1fc41331b9cdfc40d5856':'D',
              '8cbd4cfebc9ddf583a108de1a69df088':'O',
              '68d763bc4c7a9b0da3828e0b77b08b64':'T',
              '3a17ebebf2bad9aa0dd75b37a58fe6ea':'H',
              '78de2d97da222954cce639cc4b481050':'G',
              '0df9b4e759512f36aaa5c7fd4fb1fba8':'V',
              '66975492b6a53cc9a4503c3a1295b6a7':'W',
              '4a3af0b7397584c4d450c6f7e83076aa':'M',
              '2190a721b2dcb17ff693aa5feecb3b58':'P',
              'fb78aed37621262392a4125183d1bfc9':'K',
              '293f56083c20759d275db846c8bfb03e':'Z',
              '2fc20e9a20605b988999e836301a2408':'X',
              'fbe86a428051747607a35b44b1a3e9e9':'{',
              '5ae172c9ea46594cea34ad1a4b1c79cd':'J',
              'c53ba24fbbe9e3dbdd6062b3aab7ed1a':'}',
              'a94f49727cf771a85831bd03af1caaf5':'_'
              }

# Combine to form the original ciphertext
original_words = ''.join([word_dict[word] for word in word_list])

print(original_words)
```

Obtained the following output.

```
FREQUENCY ANALYSIS IS BASED ON THE FACT THAT IN ANY GIVEN STRETCH OF WRITTEN LANGUAGE CERTAIN LETTERS AND COMBINATIONS OF LETTERS OCCUR WITH VARYING FREQUENCIES MOREOVER THERE IS A CHARACTERISTIC DISTRIBUTION OF LETTERS THAT IS ROUGHLY THE SAME FOR ALMOST ALL SAMPLES OF THAT LANGUAGE IN CRYPTANALYSIS FREQUENCY ANALYSIS ALSO KNOWN AS COUNTING LETTERS IS THE STUDY OF THE FREQUENCY OF LETTERS OR GROUPS OF LETTERS IN A CIPHERTEXT THE METHOD IS USED AS AN AID TO BREAKING CLASSICAL CIPHERS FREQUENCY ANALYSIS REQUIRES ONLY A BASIC UNDERSTANDING OF THE STATISTICS OF THE PLAINTEXT LANGUAGE AND SOME PROBLEM SOLVING SKILLS AND IF PERFORMED BY HAND TOLERANCE FOR EXTENSIVE LETTER BOOKKEEPING DURING WORLD WAR II BOTH THE BRITISH AND THE AMERICANS RECRUITED CODEBREAKERS BY PLACING CROSSWORD PUZZLES IN MAJOR NEWSPAPERS AND RUNNING CONTESTS FOR WHO COULD SOLVE THEM THE FASTEST SEVERAL OF THE CIPHERS USED BY THE AXIS POWERS WERE BREAKABLE USING FREQUENCY ANALYSIS FOR EXAMPLE SOME OF THE CONSULAR CIPHERS USED BY THE JAPANESE MECHANICAL METHODS OF LETTER COUNTING AND STATISTICAL ANALYSIS GENERALLY HTB{A_SIMPLE_SUBSTITUTION_IS_WEAK} CARD TYPE MACHINERY WERE FIRST USED IN WORLD WAR II POSSIBLY BY THE US ARMYS SIS TODAY THE HARD WORK OF LETTER COUNTING AND ANALYSIS HAS BEEN REPLACED BY COMPUTER SOFTWARE WHICH CAN CARRY OUT SUCH ANALYSIS IN SECONDS WITH MODERN COMPUTING POWER CLASSICAL CIPHERS ARE UNLIKELY TO PROVIDE ANY REAL PROTECTION FOR CONFIDENTIAL DATA PUZZLE PUZZLE PUZZLE
```

### Flag

`HTB{A_SIMPLE_SUBSTITUTION_IS_WEAK}`
