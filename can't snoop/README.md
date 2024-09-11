# can't snoop
## Challenge Overview
We were given `wgmy.pcapng` to analyze. I tried to analyze protocols inside capture file using `Protocol Hierarchy`. We can see huge amount of `QUIC IETF` packets, alongside `TCP`, `TLS` and `HTTP` packets. 

Actually, I am not very familiar with `QUIC IETF` so I tried to research it a bit. 
> QUIC (Quick UDP Internet Connections) is a transport layer protocol that is designed to provide faster, more reliable internet connections by reducing latency and improving security, primarily for web traffic. It is like a more secure and faster version of TCP but utilizing UDP.

I dont know where to start from here, so I tried to seacrh for any CTF writeup related to `QUIC IETF` but that didnt return any result. Maybe we can just go through the `TCP` packets. Must of the packets are gibberish due to TLS encryption. But, there are few streams that have more readable strings from `TCP` stream 12 until 16.

Stream 12 seems like a communication between server and client. The server starts the communication with `<HLO> 00 00 00 00` and the client responds with `<HLO> 00 00 03 eb`. 

```
00000000  48 4c 4f 00 00 00 00                               HLO....
  00000000  48 4c 4f 00 00 03 eb                               HLO....
```
Next, there is `EX1` which I assume meant to be an expression followed by `K3=RC4(K,K2)`. RC4 (Rivest Cipher 4) is an encryption that XORed keytream with a plaintext to produce ciphertext, making it a basic form of symmetric encryption. You can also noticed that every there is `00 00 03 eb` after `<HLO>` and `<EX1>`. This maybe can be infered as `client_id`.

There is also another expression `EX2` which contains `00 00 00 20 85 2a 94 8d 19 69 b0 3f 4b 72 ea d7 88  20 bf 64 75 58 f9 46 cc 94 9e f7 97 63 0d b9 85 4f 08 6f`. First 4 bytes (00 00 00 20) looks like length for `K2` which is `32` bytes. 

```
00000000  45 58 31 00 00 03 eb 00  00 00 0c 4b 33 3d 52 43   EX1..... ...K3=RC
00000010  34 28 4b 2c 4b 32 29                               4(K,K2)
    00000000  45 58 32 00 00 03 eb 00  00 00 20 85 2a 94 8d 19   EX2..... .. .*...
    00000010  69 b0 3f 4b 72 ea d7 88  20 bf 64 75 58 f9 46 cc   i.?Kr...  .duX.F.
    00000020  94 9e f7 97 63 0d b9 85  4f 08 6f                  ....c... O.o
```
`EX3` followed the same format as `EX2`,  `00 00 03 eb` (client_id) followed by `00 00 00 1e f6 14 52 c2 39 5c 2f 6a a3 17 fc 7a 6b 27 19 6e 90 83 a1 67 27 cb 81 05 14 99 53 08 50 86`. The length for `K3` is `00 00 00 1e` (30 bytes).  The server reply with `00 00 03 eb 00 00 00 02 27 d3`.

```
00000000  45 58 33 00 00 03 eb 00  00 00 1e f6 14 52 c2 39   EX3..... .....R.9
00000010  5c 2f 6a a3 17 fc 7a 6b  27 19 6e 90 83 a1 67 27   \/j...zk '.n...g'
00000020  cb 81 05 14 99 53 08 50  86                        .....S.P .
    00000000  45 58 33 00 00 03 eb 00  00 00 02 27 d3            EX3..... ...'.
```
We can use CyberChef to get `K` by converting from hex, use decrypt RC4 using this formula, `K = RC4(K3, K2)`.

The function `K = RC4(K3, K2)` implies that:

- `K3` is the ciphertext or some intermediate value produced by applying the `RC4` algorithm.
- `K2` is the key that was used to encrypt or generate the pseudorandom keystream using `RC4`.

We get `K` as `69 ff 76 8b 77 b9 b1 c6 fb 0c 09 00 56 e4 90 c0 cb 97 7d 25 78 b0 16 d6 04 04 15 1d 78 76`. Use `K` for decryption. 

![image](https://github.com/user-attachments/assets/9c89df6c-e11a-4b44-afc6-bdab6724b57e)

```
00000000  47 45 54 00 00 03 eb 00  00 00 08 0e f4 23 4e 53   GET..... .....#NS
00000010  8c 16 83                                           ...
    00000000  47 45 54 00 00 03 eb 00  00 00 27 1f ff 2f 50 06   GET..... ..'../P.
    00000010  9c 08 c7 8a cc 04 c7 c3  8d 26 f6 3b b2 f9 f0 c6   ........ .&.;....
    00000020  12 05 e5 74 db bc 79 67  77 e1 50 38 b2 66 f0 3e   ...t..yg w.P8.f.>
    00000030  fd 78                                              .x
```
![image](https://github.com/user-attachments/assets/0cc88db4-af1f-4a99-a89a-070e3f776532)

## Flag
```
wgmy{df07e73edae297a5edc637c41a119de5}
```
