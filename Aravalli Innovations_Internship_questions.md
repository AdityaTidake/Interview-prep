Listed directory StreamOn
Listed directory StreamOn
Listed directory Client
Listed directory Server
Listed directory UI
Listed directory Server-client_UI
Viewed README.md:1-288
Viewed Protocol.h:1-120
Viewed RSACrypto.h:1-111
Viewed RSACrypto.cpp:1-517
Viewed kyberKEM.h:1-103
Viewed kyberKEM.cpp:1-260
Viewed keyManager.h:1-113
Viewed keyManager.cpp:1-255
Viewed FrameEncryptor.h:1-60
Viewed FrameEncrypter.cpp:1-139
Viewed FrameDecryptor.h:1-54
Viewed FrameDecryptor.cpp:1-97
Viewed StreamServer.h:1-107
Viewed StreamClient.h:1-104
Viewed StreamClient.cpp:1-393
Viewed ServerWindow.cpp:1-545
Searched for "StreamServer::"
Searched for "HandleGetKeys"
Viewed main.cpp:1-191
Viewed Logic.cpp:1-147
Viewed Logic.cpp:1-121

# StreamOn 🔒 — Comprehensive Senior Technical Interview Question Bank & Defense Guide

This guide contains **35 unique, non-repetitive, high-value technical interview questions** derived from an analysis of the **StreamOn** repository. 

Every question includes alternate question formulations, an interviewer perspective rationale, exact code snippets and technical specifics, key bullet points to state verbally, and deeper follow-up questions.

---

## 📋 Comprehensive Interview Question Bank

---

### Q1. What is StreamOn, what core problem does it solve, and what are its primary objectives?

**Category:** Project Definition & Motivation  
**Difficulty:** Basic    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To test whether you understand the high-level purpose of your project, the security motivation behind it, and whether you can articulate a 30-second elevator pitch clearly.

**Also asked as:**
* Can you explain your project in simple terms?
* What inspired you to build a secure video streaming application in C++?
* What problem in modern video streaming does StreamOn address?

**Expected Answer:**  
StreamOn is a C++ desktop application prototype designed for real-time, end-to-end encrypted webcam video streaming. Standard live video streaming protocols often transmit frames over unencrypted channels or rely solely on classic public-key cryptography (like RSA/ECC), leaving feeds vulnerable to interception and future quantum computing decryption attacks ("Harvest Now, Decrypt Later"). 

StreamOn addresses this by implementing a **hybrid cryptographic architecture** combining **Post-Quantum Cryptography (Kyber512 via `liboqs`)** for quantum-resistant key encapsulation and **RSA-2048 (via OpenSSL)** for payload security. It features two operational modes: an HTTPS REST-based polling architecture with a native Win32 GUI (`UI/Server-client_UI`) and a low-latency UDP socket-based streaming pipeline (`Server/Logic.cpp` & `Client/Logic.cpp`).

**Important points I must mention:**
* The core problem: Preventing eavesdropping/tampering on live camera feeds and protecting session keys against future quantum attack vectors.
* Dual operational modes: HTTPS REST GUI build (`cpp-httplib` + OpenSSL) vs UDP socket pipeline (`WinSock2`).
* Hybrid Cryptography: Kyber512 KEM + RSA-2048.
* Built entirely in native C++17 using OpenCV for frame capture and rendering.

**Possible follow-up questions:**
* Why did you choose C++ instead of Python or Go for a video streaming application?

---

### Q2. What is Post-Quantum Cryptography (PQC), and why is Kyber512 necessary for video streaming security?

**Category:** Project Motivation & Security  
**Difficulty:** Basic    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To gauge your depth of understanding of modern cryptography, specifically Shor's Algorithm and the concept of "Store-Now-Decrypt-Later" (SNDL) attacks.

**Also asked as:**
* Why did you include Kyber512 when RSA already provides encryption?
* What threat does PQC mitigate that standard TLS/RSA cannot?
* What makes Kyber quantum-resistant?

**Expected Answer:**  
Classical asymmetric encryption algorithms such as RSA-2048 and Elliptic Curve Cryptography (ECC) rely on the integer factorization or discrete logarithm problems, which can be solved in polynomial time by a sufficiently powerful quantum computer using **Shor's Algorithm**. Adversaries today record and store encrypted traffic ("Store-Now-Decrypt-Later"), intending to decrypt it once quantum capability arrives.

**CRYSTALS-Kyber (Kyber512)** is a lattice-based Key Encapsulation Mechanism (KEM) based on the hardness of the Module Learning-With-Errors (M-LWE) problem, which remains computationally intractable even for quantum computers. In StreamOn, Kyber512 ensures that the initial key exchange used to set up the session secret cannot be decrypted retroactively by a quantum adversary.

**Important points I must mention:**
* **Shor's Algorithm** breaks RSA/ECC; PQC resists quantum cryptanalysis.
* **Store-Now-Decrypt-Later (SNDL)**: Encrypted streams captured today can be decrypted in the future without PQC key exchange.
* **Kyber512** is based on Module-LWE (Lattice-based cryptography) and was standardized by NIST (as ML-KEM).

**Possible follow-up questions:**
* What are the key payload sizes of Kyber512 compared to RSA-2048?

---

### Q3. Can you explain the end-to-end data flow of StreamOn from camera capture to client display?

**Category:** Project Workflow  
**Difficulty:** Basic    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To verify that you understand the request/response lifecycle, data transformation pipelines, and how hardware capture hooks into networking and display code.

**Also asked as:**
* Walk me through what happens under the hood when a single frame is captured and rendered by the client.
* How does raw video data travel from the server's webcam to the client's screen?

**Expected Answer:**  
The end-to-end workflow in the HTTPS REST architecture operates in 5 steps:

```
[Webcam] -> OpenCV Capture -> JPEG Encode -> RSA Chunk Encryption 
  -> StreamServer Buffer -> HTTP GET /frames -> Client Fetch 
  -> RSA Chunk Decryption -> JPEG Decode -> GDI / OpenCV Render
```

1. **Capture & Encoding:** `CameraCapture` captures raw BGR frames (`cv::Mat`) from the local webcam using OpenCV (`cv::VideoCapture`), and compresses them into a JPEG byte array (`std::vector<uint8_t>`).
2. **Encryption:** `FrameEncryptor` splits the JPEG byte vector into **214-byte chunks** (max plaintext size for RSA-2048 with OAEP padding) and encrypts each block into **256 bytes** using OpenSSL (`RSACrypto::EncryptBlock`), producing a contiguous ciphertext buffer.
3. **Serving:** `StreamServer` buffers the latest encrypted frame thread-safely behind a mutex (`m_frameMutex`).
4. **Client Fetch & Decryption:** `StreamClient` regularly polls `GET /frames` over HTTPS, receives the ciphertext payload, and passes it to `FrameDecryptor`, which splits the stream into 256-byte blocks and decrypts each back into raw JPEG bytes using the private key.
5. **Decoding & Display:** The client decodes the JPEG bytes back into a `cv::Mat` via `cv::imdecode` and renders it on a native Win32 window using GDI (`SetDIBitsToDevice`) in `ServerWindow.cpp`/`ClientWindow.cpp` or an OpenCV window (`cv::imshow`).

**Important points I must mention:**
* Frame pipeline: `cv::Mat` $\rightarrow$ JPEG `std::vector<uint8_t>` $\rightarrow$ RSA encrypted chunks $\rightarrow$ HTTPS transmission $\rightarrow$ RSA decryption $\rightarrow$ JPEG decode $\rightarrow$ Win32 GDI display.
* Specific block sizes: 214 bytes plaintext $\rightarrow$ 256 bytes ciphertext per RSA block.

**Possible follow-up questions:**
* Why compression to JPEG before encryption rather than encrypting raw pixels?

---

### Q4. Walk me through the complete cryptographic handshake sequence executed between client and server before streaming begins.

**Category:** Project Workflow & Security  
**Difficulty:** Intermediate    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To see if you understand key encapsulation mechanics, public key distribution, and how asymmetric keys establish session secrets.

**Also asked as:**
* How do the client and server agree on cryptographic keys before transmitting video frames?
* What endpoints are involved in the handshake, and what data is exchanged over the wire?

**Expected Answer:**  
Before video transmission starts, a 2-step handshake occurs managed by [KeyManager.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/keyManager.cpp):

```
Client                                                  Server
  |                                                       |
  | ------------ GET /keys -----------------------------> |
  | <----------- JSON: {kyber_pubkey, rsa_pubkey} -------- |  (Step 1)
  |                                                       |
  | ------------ POST /handshake -----------------------> |  (Step 2)
  |              JSON: {kyber_pubkey, rsa_pubkey}         |  Encapsulates Kyber pubkey ->
  |                                                       |  Derives 32-byte shared secret
  | <----------- JSON: {kyber_ciphertext, rsa_pubkey} -- |
  | Decapsulates ciphertext using private key             |
  | -> Recovers identical 32-byte shared secret           |
```

1. **Step 1 — Public Key Fetch (`GET /keys`):** The client sends an HTTP GET request to `/keys`. The server responds with JSON containing its **Kyber512 Public Key** (800 bytes hex-encoded) and **RSA-2048 Public Key** (PEM string).
2. **Step 2 — Key Encapsulation & Exchange (`POST /handshake`):** The client generates its own local RSA and Kyber key pairs using [KeyManager::GenerateAllKeys()](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/keyManager.cpp#L21). It posts its `kyber_pubkey` and `rsa_pubkey` to `/handshake`.
3. **Server Processing:** [KeyManager::ProcessHandshakeRequest()](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/keyManager.cpp#L154) imports the client's public keys. The server runs `KyberKEM::Encapsulate()` against the client's Kyber public key, producing a **768-byte Ciphertext** and a **32-byte Shared Secret**. The server returns the `kyber_ciphertext` and its own RSA PEM to the client.
4. **Client Processing:** [KeyManager::ProcessHandshakeResponse()](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/keyManager.cpp#L203) calls `KyberKEM::Decapsulate()` with its private key and the received ciphertext, deriving the **exact same 32-byte Shared Secret**.

**Important points I must mention:**
* HTTP endpoints involved: `GET /keys` and `POST /handshake`.
* `KyberKEM::Encapsulate` runs on Server $\rightarrow$ yields Ciphertext (768B) + Shared Secret (32B).
* `KyberKEM::Decapsulate` runs on Client $\rightarrow$ recovers Shared Secret (32B).

**Possible follow-up questions:**
* How is the derived 32-byte Kyber shared secret used once the handshake finishes?

---

### Q5. What architectural patterns are used in StreamOn, and how are component responsibilities separated?

**Category:** Architecture  
**Difficulty:** Intermediate    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To test object-oriented separation of concerns, modular design, and dependency management in C++.

**Also asked as:**
* Explain the software architecture of your C++ streaming application.
* How did you structure the codebase into layers or modules?

**Expected Answer:**  
StreamOn follows a **Layered Modular Architecture** with clear Separation of Concerns (SoC) and Producer-Consumer patterns:

1. **Hardware Capture & Media Layer ([CameraCapture.h](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/CameraCapture.h) / [Recorder.h](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/Recorder.h)):** Responsible strictly for frame acquisition via OpenCV, threading, frame-rate control, and encoding/writing AVI files.
2. **Cryptographic Core Layer ([KyberKEM.h](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/kyberKEM.h), [RSACrypto.h](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/RSACrypto.h), [KeyManager.h](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/keyManager.h)):** Wraps low-level OpenSSL and liboqs C library calls into clean, object-oriented C++ classes. `KeyManager` acts as a Facade over RSA and Kyber lifecycle methods.
3. **Payload Processing Layer ([FrameEncryptor.h](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/FrameEncryptor.h) / [FrameDecryptor.h](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/FrameDecryptor.h)):** Handles chunking, padding calculations, block encryption/decryption, and vector reassembly.
4. **Networking Layer ([StreamServer.h](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/StreamServer.h) / [StreamClient.h](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/StreamClient.h)):** Encapsulates HTTP/HTTPS server endpoints and polling threads using `cpp-httplib` and WinSock2.
5. **Presentation Layer ([ServerWindow.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/ServerWindow.cpp) / [ClientWindow.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/ClientWindow.cpp)):** Native Win32 desktop UI handling event loops, GDI bitmap rendering, and status updates via Windows messages (`WM_UPDATE_FRAME`).

**Important points I must mention:**
* Clear distinction between Capture, Cryptography, Payload Transformation, Transport, and UI.
* Use of **Facade Pattern** in `KeyManager` to hide OpenSSL / liboqs complexity.
* Decoupling via callbacks (`std::function<void(const std::vector<uint8_t>&)>`) for frame passing.

---

### Q6. How is threading and concurrency designed in StreamOn? How do you prevent race conditions and UI freezing?

**Category:** Architecture & Concurrency  
**Difficulty:** Advanced    **Priority:** 🔴 High

**Why the interviewer asks this:**  
Multithreaded media applications suffer from UI blocking, data corruption, and race conditions. The interviewer wants to evaluate your multi-threading knowledge in C++17.

**Also asked as:**
* How do you avoid blocking the main UI thread during video capture and network transmission?
* Where are mutexes used in StreamOn, and what data structures do they protect?

**Expected Answer:**  
StreamOn executes across three primary concurrent thread roles:
1. **Main UI Thread:** Runs the Win32 Event Loop (`GetMessage`/`DispatchMessage`). It must **never** perform blocking I/O or crypto operations.
2. **Camera Worker Thread:** Owned by `CameraCapture`. Runs an explicit loop grabbing frames at target FPS, encoding to JPEG, and firing asynchronous callbacks.
3. **Network Server/Polling Threads:** Owned by `StreamServer` (runs `httplib::SSLServer::listen`) and `StreamClient` (runs [StreamClient::PollFrames](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/StreamClient.cpp#L325) in a loop).

**Concurrency Control & Race Condition Prevention:**
* **Mutexes:** `m_frameMutex` in `ServerWindow` and `StreamServer` protects `m_latestEncryptedFrame` during concurrent write (camera callback) and read (HTTP server route handler).
* **Atomics:** `std::atomic<bool> m_running`, `m_connected`, `m_isStreaming` handle lock-free status signaling across threads.
* **Win32 Message Posting (`PostMessage`):** When the camera thread has a new frame ready for display, it does **not** draw directly to the window DC. Instead, it copies the frame under a lock and posts `WM_UPDATE_FRAME` to the HWND queue, ensuring all GDI rendering executes strictly on the main GUI thread.

```cpp
// Thread-safe frame handoff from Camera thread to UI thread
{
    std::lock_guard<std::mutex> lock(m_frameMutex);
    m_latestFrame = frame.clone();
}
PostMessage(m_hwnd, WM_UPDATE_FRAME, 0, 0); // Asynchronous signal to UI thread
```

**Important points I must mention:**
* Separation of UI Thread, Camera Thread, and Network Threads.
* Thread-safe buffer swapping using `std::lock_guard<std::mutex>`.
* Use of Win32 `PostMessage` to offload GDI drawing onto the main UI message queue.

**Possible follow-up questions:**
* Why `PostMessage` instead of `SendMessage`? (Answer: `SendMessage` is synchronous and blocks the camera thread waiting for UI thread processing!).

---

### Q7. Why does StreamOn contain two different network implementations (HTTP Polling REST vs UDP Datagram Sockets)? Compare their trade-offs.

**Category:** Architecture & Transport Choice  
**Difficulty:** Advanced    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To test your network protocol knowledge (TCP vs UDP, HTTP Polling vs Sockets) and evaluate whether you can weigh transport trade-offs.

**Also asked as:**
* Why did you build both a UDP version and an HTTPS version in the same project?
* Which transport layer is better suited for real-time video streaming, and why?

**Expected Answer:**  
StreamOn contains two parallel implementations resulting from an architectural evolution:

1. **UDP Socket Implementation ([Server/Logic.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/Server/Logic.cpp) & [Client/Logic.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/Client/Logic.cpp)):**
   * *Protocol:* Low-level WinSock2 non-blocking UDP sockets (`SOCK_DGRAM`) with custom MTU chunking (1200-byte packets).
   * *Pros:* Low latency, minimal packet overhead, no head-of-line blocking. Ideal for real-time live streaming.
   * *Cons:* Unreliable transport (packets can drop, duplicate, or arrive out of order), no native TLS layer.

2. **HTTPS REST Implementation (`UI/Server-client_UI/`):**
   * *Protocol:* `cpp-httplib` over SSL/TLS (TCP port 8443) using HTTP GET polling (`GET /frames`).
   * *Pros:* Easily integrates with Win32 GUI, handles TLS encryption out of the box, structured JSON key exchange.
   * *Cons:* High overhead, TCP head-of-line blocking, latency penalty caused by client HTTP polling loops (`sleep_for(33ms)`).

**Architectural Assessment:**  
UDP is technically superior for live streaming because dropping an occasional frame is preferable to delaying the entire video feed due to TCP retransmissions. The HTTPS REST build served as a proof-of-concept for structured RESTful key handshakes and desktop UI integration.

**Important points I must mention:**
* UDP build focus: Low latency, custom binary protocol, MTU chunking.
* HTTPS build focus: GUI integration, REST key exchange endpoints, TLS layer.
* Trade-off: UDP gives speed/real-time delivery; HTTPS/TCP gives reliability and structured endpoints at the cost of polling latency.

---

### Q8. How is Kyber512 implemented in your codebase using `liboqs`? Explain the core C API calls.

**Category:** Implementation & Cryptography  
**Difficulty:** Intermediate    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To confirm that you didn't just read about Kyber, but actually integrated the `liboqs` C API in C++.

**Also asked as:**
* How does `KyberKEM` wrap `liboqs` in C++?
* What are the specific functions called to generate key pairs, encapsulate, and decapsulate?

**Expected Answer:**  
In [kyberKEM.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/kyberKEM.cpp), the `KyberKEM` class encapsulates `liboqs` C API functions for `OQS_KEM_alg_kyber_512`:

1. **Key Generation ([KyberKEM::GenerateKeyPair](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/kyberKEM.cpp#L31)):**
```cpp
OQS_KEM* kem = OQS_KEM_new(OQS_KEM_alg_kyber_512);
m_publicKey.resize(kem->length_public_key);   // 800 bytes
m_privateKey.resize(kem->length_secret_key); // 1632 bytes
OQS_KEM_keypair(kem, m_publicKey.data(), m_privateKey.data());
OQS_KEM_free(kem);
```
2. **Encapsulation ([KyberKEM::Encapsulate](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/kyberKEM.cpp#L170)):**
```cpp
ciphertext.resize(kem->length_ciphertext);     // 768 bytes
sharedSecret.resize(kem->length_shared_secret); // 32 bytes
OQS_KEM_encaps(kem, ciphertext.data(), sharedSecret.data(), remotePublicKey.data());
```
3. **Decapsulation ([KyberKEM::Decapsulate](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/kyberKEM.cpp#L217)):**
```cpp
sharedSecret.resize(kem->length_shared_secret); // 32 bytes
OQS_KEM_decaps(kem, sharedSecret.data(), ciphertext.data(), m_privateKey.data());
```

**Key Data Buffer Sizes (Kyber512):**
* Public Key Size: **800 bytes**
* Secret Key Size: **1632 bytes**
* Ciphertext Size: **768 bytes**
* Shared Secret Size: **32 bytes** (256 bits)

**Important points I must mention:**
* Exact API methods: `OQS_KEM_new`, `OQS_KEM_keypair`, `OQS_KEM_encaps`, `OQS_KEM_decaps`, `OQS_KEM_free`.
* Exact byte sizes for Kyber512: PubKey (800B), SecretKey (1632B), Ciphertext (768B), SharedSecret (32B).

---

### Q9. How is RSA-2048 encryption configured in OpenSSL, and why is the maximum chunk size set to 214 bytes?

**Category:** Cryptography & Mathematical Limits  
**Difficulty:** Intermediate    **Priority:** 🔴 High

**Why the interviewer asks this:**  
This is a classic cryptography math check. Interviewers want to see if you understand RSA key size vs padding overhead limits.

**Also asked as:**
* Why did you split the JPEG frame into 214-byte chunks in `RSACrypto`?
* What padding scheme is used for RSA, and how does it affect payload capacity?

**Expected Answer:**  
In [Protocol.h](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/Protocol.h#L24-L26) and [RSACrypto.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/RSACrypto.cpp#L264), RSA encryption is configured using **RSA-2048** ($2048 \text{ bits} = 256 \text{ bytes}$ block size) with **Optimal Asymmetric Encryption Padding (OAEP)** (`RSA_PKCS1_OAEP_PADDING`).

**Math Breakdown for 214 Bytes Chunk Size:**
* An RSA key of modulus size $N = 2048 \text{ bits}$ produces ciphertext blocks of exactly **256 bytes**.
* Standard PKCS#1 v1.5 padding requires 11 bytes of overhead ($256 - 11 = 245 \text{ bytes}$ max payload).
* However, OpenSSL's `RSA_PKCS1_OAEP_PADDING` using SHA-1 overhead is **42 bytes** ($2 \times \text{hash\_len} + 2 = 2 \times 20 + 2 = 42 \text{ bytes}$).
* Maximum Plaintext Size = $\text{Key Size in Bytes} - \text{Padding Overhead} = 256 - 42 = \mathbf{214 \text{ bytes}}$.

```cpp
// Encrypting block-by-block in RSACrypto.cpp
EVP_PKEY_CTX_set_rsa_padding(ctx, RSA_PKCS1_OAEP_PADDING);
// Input chunk length MUST be <= 214 bytes
// Output block length will ALWAYS be 256 bytes
```

If an unencrypted chunk exceeds 214 bytes, `EVP_PKEY_encrypt` fails with an OpenSSL buffer error.

**Important points I must mention:**
* RSA-2048 modulus size = 2048 bits = 256 bytes.
* Padding scheme used: `RSA_PKCS1_OAEP_PADDING`.
* OAEP padding overhead = 42 bytes $\implies 256 - 42 = 214$ max payload chunk size.
* Encrypted output block size is fixed at 256 bytes.

---

### Q10. Why did you use a Hybrid Cryptographic model (RSA + Kyber512) instead of using Kyber or RSA alone?

**Category:** Cryptography & Design Decisions  
**Difficulty:** Advanced    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To test whether you understand the functional distinction between a Key Encapsulation Mechanism (KEM) and an Encryption Cipher.

**Also asked as:**
* Can't you encrypt the video stream directly using Kyber512?
* Why do you need both RSA and Kyber in the handshake?

**Expected Answer:**  
Kyber is strictly a **Key Encapsulation Mechanism (KEM)**, not a general-purpose block cipher or bulk encryption algorithm. You cannot feed arbitrary video data bytes into Kyber to encrypt them. Kyber's sole purpose is to securely negotiate a shared secret key (32 bytes) between two parties over an insecure channel.

Once the shared secret is established via Kyber, data payload encryption requires a cipher. In StreamOn's GUI architecture, **RSA-2048** was combined with Kyber to demonstrate hybrid key exchange and asymmetric chunk encryption:
* **Kyber512** provides **Quantum Resistance** for establishing the session secret.
* **RSA-2048** provides **Authentication & Digital Signatures** ([RSACrypto::Sign](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/RSACrypto.cpp#L412) / `Verify`) and chunk encryption.

Combining classical primitives with PQC is recommended by NIST during the quantum transition period so that security relies on *both* hardness assumptions simultaneously.

**Important points I must mention:**
* Kyber is a **KEM**, not a bulk encryption cipher; it cannot directly encrypt video files.
* Hybrid approach combines Kyber's **Post-Quantum Key Exchange** with RSA's **Authentication & Signatures**.
* Recommended by NIST: Defense-in-depth during the PQC transition phase.

---

### Q11. How is video frame encryption implemented in the UDP build, and what is its security vulnerability?

**Category:** Cryptography & Security Vulnerabilities  
**Difficulty:** Intermediate    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To see if you can evaluate existing code critically and identify cryptanalytic weaknesses in custom ciphers.

**Also asked as:**
* How does `Server/Logic.cpp` encrypt JPEG data before sending over UDP?
* Is byte-wise XORing with a 32-byte key secure for live streaming?

**Expected Answer:**  
In the UDP implementation ([Server/Logic.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/Server/Logic.cpp#L104) and [Client/Logic.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/Client/Logic.cpp#L86)), video frame encryption uses a **Byte-wise XOR Stream Cipher** repeating the 32-byte Kyber shared secret:

```cpp
// Encrypting JPEG buffer with shared_key in Server/Logic.cpp
for (size_t i = 0; i < buf.size(); i++) {
    buf[i] ^= shared_key[i % shared_key.size()];
}
```

**Security Vulnerability (Known-Plaintext Attack):**
1. **Predictable Magic Bytes:** Standard JPEG files **always** start with fixed header bytes (`0xFF 0xD8 0xFF 0xE0` or `0xFF 0xD8 0xFF 0xE1`).
2. **Key Extraction:** An attacker eavesdropping on the network can XOR the ciphertext frame header with the known JPEG magic bytes:
   $$\text{Key}[i] = \text{Ciphertext}[i] \oplus \text{KnownJPEGHeader}[i]$$
   Because the key repeats every 32 bytes, an attacker can extract the first 4–8 bytes of the shared key immediately from every single frame.
3. **Lack of Nonce / Initialization Vector (IV):** Encrypting multiple frames with the exact same static key without an IV allows key reuse attacks across video frames.

**Inference & Verification:** This lightweight XOR cipher was implemented in the UDP prototype for microsecond execution speed, but in production, it **must** be replaced by AES-256-GCM or ChaCha20-Poly1305.

**Important points I must mention:**
* Implementation: Simple XOR loop `buf[i] ^= shared_key[i % 32]`.
* Flaw: Vulnerable to **Known-Plaintext Attacks** due to static JPEG headers (`0xFF 0xD8`).
* Lack of IV/Nonce leading to key reuse across frames.
* Production fix: AES-256-GCM or ChaCha20-Poly1305.

---

### Q12. How are cryptographic keys stored on disk, and how does StreamOn protect private keys in memory?

**Category:** Cryptography & Security  
**Difficulty:** Intermediate    **Priority:** 🟡 Medium

**Why the interviewer asks this:**  
To test key persistence patterns, file formats (PEM vs Binary), and memory sanitization practices in C++.

**Also asked as:**
* Where are RSA and Kyber keys saved, and in what format?
* How do you clean up sensitive cryptographic keys when the program exits?

**Expected Answer:**  
Key storage and memory lifecycle are managed in [KeyManager.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/keyManager.cpp) and [kyberKEM.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/kyberKEM.cpp):

1. **Storage Formats on Disk:**
   * **RSA-2048 Keys:** Saved as text files in Privacy-Enhanced Mail (**PEM**) format (`server_rsa_private.pem`, `server_rsa_public.pem`) using OpenSSL BIO functions (`PEM_write_bio_PrivateKey`, `PEM_write_bio_PUBKEY`).
   * **Kyber512 Keys:** Saved as raw **binary** files (`server_kyber_private.bin`, `server_kyber_public.bin`) because standard PEM headers are not natively defined for raw PQC key buffers.

2. **In-Memory Sanitization (Secure Cleanup):**
   Standard C++ `std::vector` destructors deallocate memory without overwriting contents, leaving sensitive private key bytes sitting in RAM where memory dumpers or cold-boot attacks can retrieve them.
   
   In `KyberKEM`'s destructor and byte loaders, `liboqs`'s **`OQS_MEM_cleanse()`** function is explicitly called to securely zero out private key memory before releasing the buffer:

```cpp
KyberKEM::~KyberKEM() {
    if (!m_privateKey.empty()) {
        OQS_MEM_cleanse(m_privateKey.data(), m_privateKey.size()); // Zero-fill memory
    }
}
```

**Important points I must mention:**
* RSA saved in PEM format; Kyber saved as raw binary files.
* Use of `OQS_MEM_cleanse` to zero-fill memory buffers containing secret keys to prevent memory scraping.

---

### Q13. How does transport layer security (TLS/HTTPS) work in `StreamServer` and `StreamClient`, and how are certificates handled?

**Category:** Security & Networking  
**Difficulty:** Intermediate    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To verify your understanding of SSL/TLS certificate verification, self-signed certificates, and `cpp-httplib` configuration.

**Also asked as:**
* Why does `StreamClient` disable certificate verification in `Connect()`?
* How is `httplib::SSLServer` initialized with certs?

**Expected Answer:**  
In `StreamServer.h` and `StreamClient.cpp`, HTTPS server/client instances are initialized using `cpp-httplib` compiled with OpenSSL flags (`CPPHTTPLIB_OPENSSL_SUPPORT`).

1. **Server-Side TLS Setup:**
   `StreamServer` initializes `httplib::SSLServer` by passing paths to a TLS X.509 Certificate (`server.crt`) and RSA Private Key (`server.key`):
```cpp
m_server = new httplib::SSLServer(TLS_CERT_FILE, TLS_KEY_FILE);
```
2. **Client-Side Certificate Handling ([StreamClient.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/StreamClient.cpp#L84-L85)):**
```cpp
m_client = new httplib::SSLClient(serverHost, serverPort);
m_client->enable_server_certificate_verification(false); // Disables CA chain check
```

**Trade-off & Insecure Inferred Configuration:**
In this local development setup, `enable_server_certificate_verification(false)` is set to allow testing with **self-signed certificates** without installing a custom Root CA certificate in the Windows Certificate Store. 

**Production Requirement:** In production, certificate verification **must** be enabled, and the client must pin the CA root or server certificate to prevent Man-In-The-Middle (MITM) attacks.

**Important points I must mention:**
* `httplib::SSLServer` uses OpenSSL underlying contexts initialized with `.crt` and `.key`.
* `enable_server_certificate_verification(false)` bypasses self-signed cert errors for local testing.
* Highlight MITM vulnerability if certificate verification remains disabled in production.

---

### Q14. How does the camera capture layer work in OpenCV, and how do you control frame rates and JPEG compression quality?

**Category:** Media Pipeline  
**Difficulty:** Basic    **Priority:** 🟡 Medium

**Why the interviewer asks this:**  
To test your knowledge of OpenCV video acquisition, DirectShow backends on Windows, and image compression parameters.

**Also asked as:**
* How do you capture frames from a webcam in C++?
* How does changing JPEG compression quality affect network bandwidth and CPU usage?

**Expected Answer:**  
Camera capture is implemented in [Server/Logic.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/Server/Logic.cpp#L75) and `CameraCapture.cpp`:

1. **DirectShow Backend Capture:**
   On Windows, `cv::VideoCapture` is opened specifying the DirectShow backend (`CAP_DSHOW`):
```cpp
cap.open(0, cv::CAP_DSHOW); // Hardware direct capture on camera device index 0
```
2. **JPEG Compression Tuning:**
   Raw camera frames captured as uncompressed `cv::Mat` (640x480 resolution, BGR color space) require $640 \times 480 \times 3 = 921,600 \text{ bytes}$ (~920 KB) per frame. Streaming raw uncompressed frames over network or encrypting them chunk-by-chunk would crush CPU and bandwidth.

   StreamOn compresses frames to JPEG using `cv::imencode`:
```cpp
std::vector<uchar> buf;
std::vector<int> params = { cv::IMWRITE_JPEG_QUALITY, 45 }; // Set compression quality
cv::imencode(".jpg", smallerFrame, buf, params);
```
Setting `IMWRITE_JPEG_QUALITY` to **45%** reduces frame size from **920 KB down to ~20–30 KB** (~97% compression ratio), drastically decreasing the number of RSA chunks or UDP packets required per frame.

**Important points I must mention:**
* Uncompressed frame size = ~920 KB ($640 \times 480 \times 3$).
* `cv::imencode(".jpg", ...)` reduces size to 20–30 KB.
* `IMWRITE_JPEG_QUALITY` set to 45% balances visual clarity against transmission size.

---

### Q15. How does packet chunking, MTU fragmentation, and header protocol design work in the UDP streaming build?

**Category:** Networking & Protocol Design  
**Difficulty:** Intermediate    **Priority:** 🟡 Medium

**Why the interviewer asks this:**  
To test your understanding of UDP Maximum Transmission Unit (MTU) restrictions and binary header serialization in C++.

**Also asked as:**
* How do you prevent IP fragmentation when sending large video frames over UDP?
* What is the binary structure of the UDP packet header in `Server/Logic.cpp`?

**Expected Answer:**  
Standard Ethernet networks have a Maximum Transmission Unit (**MTU**) of **1500 bytes**. Transmitting a UDP packet larger than 1472 bytes (1500 MTU - 20B IP header - 8B UDP header) causes IP-level fragmentation, which significantly increases packet loss rates.

In [Server/Logic.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/Server/Logic.cpp#L122-L139), StreamOn implements **Application-Layer MTU Fragmentation**:

1. **Chunk Size Setting:** Max payload chunk size per packet is set to **1200 bytes** (`chunkSize = 1200`), safely below the 1472 MTU limit.
2. **9-Byte Custom Header Serialization:** Every UDP packet contains a 9-byte header prepended to the 1200-byte encrypted frame slice:

| Offset (Bytes) | Field Name | Type | Size | Description |
| :--- | :--- | :--- | :--- | :--- |
| `0` | Packet Type | `uint8_t` | 1 Byte | `0x03` = `FRAME_DATA` |
| `1 – 4` | Frame ID | `uint32_t` | 4 Bytes | Monotonically increasing frame index (Big-Endian bitshifts) |
| `5 – 6` | Total Chunks | `uint16_t` | 2 Bytes | Total number of chunks constructing this frame |
| `7 – 8` | Chunk Index | `uint16_t` | 2 Bytes | Zero-based index of current chunk (`0` to `TotalChunks - 1`) |
| `9+` | Payload Data | `uint8_t[]` | Variable | Encrypted JPEG byte slice (max 1200 bytes) |

```cpp
// Header construction using manual big-endian bit shifts
vector<uint8_t> pkt(9 + length);
pkt[0] = 3; // Packet type 3 = Frame Data
pkt[1] = (uint8_t)((frameId >> 24) & 0xFF); 
pkt[2] = (uint8_t)((frameId >> 16) & 0xFF); 
pkt[3] = (uint8_t)((frameId >> 8) & 0xFF);  
pkt[4] = (uint8_t)(frameId & 0xFF);
pkt[5] = (uint8_t)((totalChunks >> 8) & 0xFF); 
pkt[6] = (uint8_t)(totalChunks & 0xFF);
pkt[7] = (uint8_t)((chunkIdx >> 8) & 0xFF); 
pkt[8] = (uint8_t)(chunkIdx & 0xFF);
memcpy(&pkt[9], &buf[offset], length);
```

**Important points I must mention:**
* MTU limitation: Ethernet 1500 bytes; application payload kept to 1200 bytes.
* 9-byte explicit binary header layout: `[PktType (1B)][FrameID (4B)][TotalChunks (2B)][ChunkIndex (2B)]`.
* Use of bitwise shifts (`>> 24`, `>> 16`, `>> 8`) for endianness safety.

---

### Q16. How does the desktop UI layer display video frames on Windows, and why did you use GDI (`SetDIBitsToDevice`) instead of standard OpenCV windows in the GUI build?

**Category:** UI Architecture & Win32  
**Difficulty:** Intermediate    **Priority:** 🟡 Medium

**Why the interviewer asks this:**  
To evaluate your native Windows API (Win32 GDI) knowledge vs generic highgui frameworks like OpenCV `imshow`.

**Also asked as:**
* How do you draw a OpenCV `cv::Mat` inside a native Win32 window handle (`HWND`)?
* What is `SetDIBitsToDevice` doing in `ServerWindow.cpp`?

**Expected Answer:**  
While `cv::imshow` is convenient for fast prototyping, it creates separate unmanaged top-level windows that cannot be embedded into professional Win32 window controls, dialogs, or custom UI layouts.

In [ServerWindow.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/ServerWindow.cpp#L371-L408), video frames are rendered directly into a Win32 static control handle (`m_staticVideo`) using **GDI (Graphics Device Interface)**:

1. **Color Space Conversion:** OpenCV stores images in BGR format. To render via GDI, the frame is resized to fit the static control rect and converted to 32-bit BGRA:
```cpp
cv::cvtColor(resized, bgra, cv::COLOR_BGR2BGRA);
```
2. **BITMAPINFO Header Setup:** A `BITMAPINFO` structure is populated describing a top-down 32-bit uncompressed bitmap:
```cpp
BITMAPINFO bmi = {};
bmi.bmiHeader.biSize = sizeof(BITMAPINFOHEADER);
bmi.bmiHeader.biWidth = bgra.cols;
bmi.bmiHeader.biHeight = -bgra.rows; // Negative height indicates top-down DIB
bmi.bmiHeader.biPlanes = 1;
bmi.bmiHeader.biBitCount = 32;
bmi.bmiHeader.biCompression = BI_RGB;
```
3. **Blitting to Screen Device Context (`SetDIBitsToDevice`):**
```cpp
HDC panelDC = GetDC(m_staticVideo);
SetDIBitsToDevice(panelDC, 0, 0, bgra.cols, bgra.rows, 0, 0, 0, bgra.rows, bgra.data, &bmi, DIB_RGB_COLORS);
ReleaseDC(m_staticVideo, panelDC);
```

**Important points I must mention:**
* Reason for GDI: Embedding live video natively inside Win32 `HWND` controls instead of pop-up `cv::imshow` windows.
* Use of negative `biHeight` for top-down device-independent bitmap (DIB) orientation.
* Color conversion from BGR to 32-bit BGRA (`COLOR_BGR2BGRA`).

---

### Q17. How is video recording implemented in StreamOn, and what container format / codec is used?

**Category:** Media Pipeline  
**Difficulty:** Intermediate    **Priority:** 🟢 Low

**Why the interviewer asks this:**  
To test your understanding of video multiplexing, FourCC codes, and synchronous frame writing to disk.

**Also asked as:**
* How does the `Recorder` class capture video while streaming is active?
* What FourCC codec was selected for recording, and why?

**Expected Answer:**  
Video recording is encapsulated in `Recorder.cpp` and [ServerWindow.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/ServerWindow.cpp#L305-L341):

1. **Initialization:** When the user clicks "Record", `Recorder::StartRecording()` creates an output filename with a timestamp (`record_YYYYMMDD_HHMMSS.avi`).
2. **Codec Selection (MJPEG):** The video writer is initialized using OpenCV's `cv::VideoWriter` with the **MJPG FourCC code** (`cv::VideoWriter::fourcc('M','J','P','G')`) at 20 FPS:
```cpp
data.writer.open(filename, cv::VideoWriter::fourcc('M','J','P','G'), 20.0, frameSize, true);
```
3. **Dual Pipeline Feeding:** In the camera capture callback, frames are decoded and simultaneously fed to both the network streaming pipeline and the local video recorder:
```cpp
m_server.FeedFrame(jpegBytes);
if (m_isRecording.load()) {
    m_recorder.FeedFrame(jpegBytes); // Writes uncompressed frame to VideoWriter
}
```

**Trade-off:** Writing frames synchronously on the main callback thread introduces file I/O latency. In production, `Recorder` should use an internal lock-free queue and worker thread for background disk writing.

**Important points I must mention:**
* FourCC code used: `'M','J','P','G'` (Motion JPEG) in an `.avi` container.
* Simultaneous dual-feeding of camera frames to network server and video writer.

---

### Q18. Explain the design and state machine of the `KeyManager` class.

**Category:** Code Implementation  
**Difficulty:** Intermediate    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To assess your object-oriented design skills and state management capabilities in C++.

**Also asked as:**
* What does `KeyManager` do, and how does it keep track of local vs remote public keys?
* What are the internal state flags inside `KeyManager`?

**Expected Answer:**  
[KeyManager.h](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/keyManager.h) acts as an orchestration facade over `RSACrypto` and `KyberKEM`. It manages key generation, disk persistence, and key state during the handshake:

**Internal State Variables:**
* `m_localRSA`: Instance of `RSACrypto` holding **OUR** private key and public key.
* `m_remoteRSA`: Instance of `RSACrypto` holding the **REMOTE PARTY's** public key (loaded during handshake).
* `m_kyber`: Instance of `KyberKEM` managing local Kyber keys and encapsulation/decapsulation.
* `m_handshakeComplete` (`bool`): Set to `true` only after successful key decapsulation/encapsulation.
* `m_hasLocalKeys` (`bool`): Set to `true` when RSA/Kyber keys are generated or loaded from disk.

**Key Methods:**
* [GenerateAllKeys()](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/keyManager.cpp#L21): Generates fresh RSA-2048 and Kyber512 keypairs and saves them to disk.
* [ProcessHandshakeRequest()](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/keyManager.cpp#L154): Executed on Server. Loads Client's RSA key into `m_remoteRSA`, encapsulating Kyber shared secret.
* [ProcessHandshakeResponse()](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/keyManager.cpp#L203): Executed on Client. Loads Server's RSA key into `m_remoteRSA`, decapsulating Kyber ciphertext.

**Important points I must mention:**
* Role of `KeyManager`: Coordinates local RSA/Kyber instances and remote RSA instance.
* Key isolation: `m_localRSA` has private keys (for decryption); `m_remoteRSA` has public keys (for encryption).

---

### Q19. How does `RSACrypto` manage OpenSSL C API memory using BIO abstractions without leaking memory?

**Category:** C++ / Resource Management  
**Difficulty:** Intermediate    **Priority:** 🟡 Medium

**Why the interviewer asks this:**  
OpenSSL C APIs rely on manual raw pointer allocations (`EVP_PKEY*`, `BIO*`). The interviewer wants to verify that you follow proper RAII principles in C++.

**Also asked as:**
* How do you convert an `EVP_PKEY` public key into a PEM string using OpenSSL BIOs?
* How does `RSACrypto` prevent memory leaks when loading or freeing RSA keys?

**Expected Answer:**  
OpenSSL 3.x uses raw opaque pointers (`EVP_PKEY*`, `EVP_PKEY_CTX*`, `BIO*`) that require explicit memory deallocation. In [RSACrypto.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/RSACrypto.cpp#L183-L206), memory management and string serialization are handled safely using OpenSSL **In-Memory BIOs (`BIO_s_mem`)**:

**Exporting Public Key to PEM String ([RSACrypto::GetPublicKeyPEM](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/RSACrypto.cpp#L183)):**
```cpp
BIO* bio = BIO_new(BIO_s_mem()); // Allocate memory stream BIO
if (PEM_write_bio_PUBKEY(bio, key) > 0) {
    char* data = nullptr;
    long len = BIO_get_mem_data(bio, &data); // Read raw string pointer from BIO
    std::string pem(data, static_cast<size_t>(len)); // Construct C++ std::string
    BIO_free(bio); // Free BIO memory buffer immediately
    return pem;
}
BIO_free(bio);
return "";
```

**RAII Destructor Cleanup ([RSACrypto::~RSACrypto](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/RSACrypto.cpp#L16-L23)):**
```cpp
RSACrypto::~RSACrypto() {
    if (m_publicKey)  EVP_PKEY_free(m_publicKey);  // Decrement ref-count & free
    if (m_privateKey) EVP_PKEY_free(m_privateKey); // Decrement ref-count & free
}
```

**Important points I must mention:**
* Use of `BIO_s_mem()` memory buffers to bridge OpenSSL C APIs with C++ `std::string`.
* Manual cleanup using `BIO_free()`, `EVP_PKEY_free()`, and `EVP_PKEY_CTX_free()` to eliminate memory leaks.

---

### Q20. How does `FrameEncryptor` transform raw JPEG bytes into encrypted payload packets?

**Category:** Code Implementation  
**Difficulty:** Intermediate    **Priority:** 🟡 Medium

**Why the interviewer asks this:**  
To test your comprehension of chunk slicing loops, vector pre-allocations, and error handling during streaming payload generation.

**Also asked as:**
* Explain the step-by-step logic inside `FrameEncryptor::EncryptFrame()`.
* How are empty frames or RSA encryption failures handled?

**Expected Answer:**  
In [FrameEncrypter.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/FrameEncrypter.cpp#L32-L84), `FrameEncryptor` performs chunking and block assembly:

```cpp
std::vector<uint8_t> FrameEncryptor::EncryptFrame(const std::vector<uint8_t>& jpegBytes) {
    std::lock_guard<std::mutex> lock(m_mutex);
    if (!m_rsa || !m_rsa->HasPublicKey() || jpegBytes.empty()) return {};

    // Step 1: Split JPEG vector into 214-byte chunks
    auto chunks = SplitIntoChunks(jpegBytes, RSA_MAX_CHUNK_SIZE);

    // Step 2: Pre-allocate continuous output vector memory
    std::vector<uint8_t> encrypted;
    encrypted.reserve(chunks.size() * RSA_BLOCK_SIZE); // Reserve exact size (N * 256 bytes)

    // Step 3: Encrypt each chunk and append to contiguous vector
    for (size_t i = 0; i < chunks.size(); i++) {
        auto encBlock = m_rsa->EncryptBlock(chunks[i]);
        if (encBlock.empty()) return {}; // Fail fast if encryption errors out
        AppendBytes(encrypted, encBlock); // Append 256 bytes block
    }

    m_frameCount.fetch_add(1); // Increment atomic frame counter
    return encrypted;
}
```

**Memory Optimization:** `encrypted.reserve(chunks.size() * RSA_BLOCK_SIZE)` allocates the exact necessary memory up front, preventing multiple dynamic memory re-allocations during `std::vector::insert`.

**Important points I must mention:**
* Input validation: Null RSA pointer check, public key state check, empty frame check.
* Vector memory pre-allocation using `reserve()` to avoid intermediate buffer copies.
* Atomic counter increment (`m_frameCount.fetch_add(1)`).

---

### Q21. How does `FrameDecryptor` validate ciphertext payload integrity and reconstruct original JPEG frames?

**Category:** Code Implementation  
**Difficulty:** Intermediate    **Priority:** 🟡 Medium

**Why the interviewer asks this:**  
To see how you implement defensive checks (bounds checking, alignment validation) on incoming network buffers before decrypting.

**Also asked as:**
* What sanity checks does `FrameDecryptor` run before decrypting bytes?
* What happens if a corrupted network frame is fed to `FrameDecryptor`?

**Expected Answer:**  
In [FrameDecryptor.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/FrameDecryptor.cpp#L32-L97), `FrameDecryptor` enforces defensive validation before decryption:

1. **RSA Block Alignment Check:** Since every RSA block **must** be exactly 256 bytes, any ciphertext vector whose total length is not a multiple of 256 bytes is rejected immediately:
```cpp
if (encryptedBytes.size() % RSA_BLOCK_SIZE != 0) {
    LogError("DecryptFrame: invalid payload size - not multiple of 256");
    return {}; // Reject malformed payload
}
```
2. **Chunk Splitting & Block Decryption:** The ciphertext is sliced into 256-byte blocks using `SplitIntoChunks(encryptedBytes, 256)`.
3. **Block-by-Block RSA Decryption:** Each 256-byte block is decrypted using `RSACrypto::DecryptBlock()`, which removes the OAEP padding and returns original plaintext bytes (up to 214 bytes per block).
4. **Reassembly & Image Decoding:** The decrypted chunks are concatenated into a contiguous `std::vector<uint8_t>` containing valid JPEG file bytes (`0xFF 0xD8 ... 0xFF 0xD9`), which is subsequently passed to `cv::imdecode`.

**Important points I must mention:**
* Defensive check: `encryptedBytes.size() % 256 == 0`.
* Fail-fast strategy: If any single block fails OAEP padding check, `DecryptFrame` aborts and drops the corrupted frame.

---

### Q22. How is `cpp-httplib` integrated for HTTPS networking, and how are HTTP endpoints declared?

**Category:** Implementation & Networking  
**Difficulty:** Intermediate    **Priority:** 🟡 Medium

**Why the interviewer asks this:**  
To evaluate your experience with modern header-only C++ HTTP libraries and lambda-based route registration.

**Also asked as:**
* How do you create an HTTPS server in C++ using `cpp-httplib`?
* Explain the endpoint definitions in `StreamServer`.

**Expected Answer:**  
In `StreamServer.h` and `StreamServer.cpp`, StreamOn uses `cpp-httplib` (with OpenSSL support) to set up a multi-threaded HTTPS server.

**Endpoint Definitions:**
* `GET /keys`: Serves server public keys as JSON.
* `POST /handshake`: Accepts client public keys, runs Kyber encapsulation, and returns ciphertext.
* `GET /frames`: Serves the latest RSA-encrypted JPEG frame.

**Route Registration Example:**
```cpp
m_server->Get("/keys", [this](const httplib::Request& req, httplib::Response& res) {
    HandleGetKeys(req, res);
});

m_server->Get("/frames", [this](const httplib::Request& req, httplib::Response& res) {
    std::lock_guard<std::mutex> lock(m_frameMutex);
    if (m_latestEncryptedFrame.empty()) {
        res.status = 204; // HTTP 204 No Content
        return;
    }
    res.set_content(reinterpret_cast<const char*>(m_latestEncryptedFrame.data()), 
                    m_latestEncryptedFrame.size(), "application/octet-stream");
});
```

**Thread Execution Model:** `httplib::SSLServer::listen("localhost", 8443)` is spawned inside a dedicated background thread (`m_serverThread = std::thread(...)`), allowing the server to process concurrent HTTP requests without blocking the GUI.

**Important points I must mention:**
* Use of `httplib::SSLServer` running in a dedicated `std::thread`.
* Returns HTTP `204 No Content` when no new video frame is buffered.
* Binary payload serving with `application/octet-stream` Content-Type.

---

### Q23. Why is using RSA-2048 for frame-by-frame video payload encryption a major technical anti-pattern/trade-off, and how should it be fixed?

**Category:** Architectural Trade-offs & Design Errors  
**Difficulty:** Advanced    **Priority:** 🔴 High

**Why the interviewer asks this:**  
This is a critical senior-level interview question. RSA is asymmetric math and is **never** used for bulk payload encryption in production. The interviewer wants to verify that you recognize this design trade-off and know the correct solution.

**Also asked as:**
* What is the biggest performance bottleneck in StreamOn's GUI encryption pipeline?
* Why is RSA impractical for real-time 60 FPS video frame encryption?

**Expected Answer:**  
Encrypting frame payload data directly with RSA-2048 is a severe technical anti-pattern for two primary reasons:

1. **Massive Computational Overhead (CPU Bottleneck):**
   RSA encryption involves modular exponentiations ($C = M^e \pmod N$) on 2048-bit numbers. Slicing a single 30 KB JPEG frame into 140 individual 214-byte chunks requires **140 modular exponentiations per frame**. At 30 FPS, the CPU must compute $140 \times 30 = \mathbf{4,200 \text{ RSA operations/sec}}$, saturating CPU cores and causing low frame rates.
2. **Data Expansion Overhead:**
   Every 214 bytes of plaintext expands to 256 bytes of ciphertext—a **~19.6% payload expansion penalty** on bandwidth.

**The Production Fix (Symmetric Hybrid Encryption Scheme):**
Asymmetric ciphers (RSA/Kyber) should **only** be used during the initial handshake to exchange a 256-bit symmetric session key. All video frame payloads must subsequently be encrypted using a high-speed symmetric cipher like **AES-256-GCM** or **ChaCha20-Poly1305** (which leverages hardware AES-NI instructions):

```
Handshake (Once):    Kyber / RSA  ---> Derive 256-bit Symmetric Key
Video Stream (FPS):  AES-256-GCM (Symmetric) ---> Encrypts entire 30 KB frame in < 10 microseconds!
```

**Inference:** RSA frame chunking was implemented in StreamOn as an experimental prototype mechanism to demonstrate OpenSSL chunking loops, but converting to AES-GCM is the #1 priority for production readiness.

**Important points I must mention:**
* RSA modular exponentiation is too computationally expensive for real-time video payload processing ($4200 \text{ ops/sec}$ at 30 FPS).
* Payload expansion penalty (~19.6%).
* Industry standard solution: Use RSA/Kyber **only** for session key agreement; use **AES-256-GCM** or **ChaCha20-Poly1305** with AES-NI hardware acceleration for stream payloads.

---

### Q24. What are the engineering trade-offs between HTTP Polling (`GET /frames`) vs WebSockets vs RTP/RTSP for live streaming?

**Category:** Architectural Trade-offs & Scalability  
**Difficulty:** Advanced    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To test your knowledge of media streaming protocols and client-server communication models.

**Also asked as:**
* Why is HTTP polling inefficient for live camera streams?
* How would you redesign StreamOn's transport layer to support low-latency streaming?

**Expected Answer:**  

| Metric / Transport | HTTP Polling (`GET /frames`) [StreamOn GUI] | WebSockets / WebRTC | RTP/RTSP over UDP [StreamOn UDP] |
| :--- | :--- | :--- | :--- |
| **Communication Model** | Client Pull (Polling Loop) | Full-Duplex Push | Server Push (Datagram Stream) |
| **Protocol Overhead** | High (HTTP Headers per request) | Low (Framed Byte Masking) | Minimal (12-byte RTP Header) |
| **Latency** | High (100 ms – 500 ms penalty) | Low (< 50 ms) | Ultra-Low (< 10 ms) |
| **Head-of-Line Blocking** | Severe (TCP-level blocking) | Moderate (TCP-level) | None (UDP Datagrams) |
| **Network Efficiency** | Very Poor (Wasted HTTP requests) | Good | Excellent |

**Why HTTP Polling is Inefficient:**
In [StreamClient::PollFrames](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/StreamClient.cpp#L325), the client continuously executes `m_client->Get("/frames")` inside a while loop with `sleep_for(33ms)`. This generates thousands of redundant HTTP GET request/response header overheads, wastes CPU cycles, and risks frame stuttering if HTTP connection setup stalls.

**Production Redesign:** Replace HTTP polling with **WebSockets (TLS)** or **RTP/SRTP over UDP**, allowing the server to push newly captured frames to the client instantly as soon as they are encoded.

**Important points I must mention:**
* HTTP Polling creates massive HTTP header overhead and latency penalties.
* WebSockets or RTP (Real-Time Transport Protocol) over UDP are industry standards for low-latency live video push streaming.

---

### Q25. How does StreamOn handle frame buffering, and what strategy is used when the network falls behind camera capture?

**Category:** Concurrency & Network Performance  
**Difficulty:** Advanced    **Priority:** 🟡 Medium

**Why the interviewer asks this:**  
To see how your application handles backpressure when production networks experience congestion.

**Also asked as:**
* What happens if the network is slower than the camera capture frame rate (e.g., 60 FPS camera, 10 FPS network)?
* Do you drop frames or queue them in memory?

**Expected Answer:**  
StreamOn implements a **Latest-Frame Overwrite (Frame-Dropping) Strategy** rather than an unbounded frame queue:

In [StreamServer.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/StreamServer.h#L97), `StreamServer` maintains a single vector buffer (`m_latestEncryptedFrame`) protected by `m_frameMutex`:

```cpp
void StreamServer::FeedFrame(const std::vector<uint8_t>& jpegBytes) {
    auto encrypted = m_frameEncryptor->EncryptFrame(jpegBytes);
    {
        std::lock_guard<std::mutex> lock(m_frameMutex);
        m_latestEncryptedFrame = std::move(encrypted); // Overwrites previous frame in-place!
    }
}
```

**Trade-off Analysis:**
* **Pros (Real-time Focus):** If a client polls slowly due to network congestion, `StreamServer` simply overwrites the internal buffer with the newest camera frame. When the client finally requests `/frames`, it gets the **most recent** live frame rather than stale historical frames. Memory consumption remains strictly bounded $O(1)$.
* **Cons (Frame Rate Degradation):** Older intermediate frames are silently dropped, causing video playback to appear choppy during network congestion.

**Important points I must mention:**
* Single-frame buffer overwrite strategy (`m_latestEncryptedFrame = std::move(...)`).
* Bounded memory footprint (prevents out-of-memory crashes under network lag).
* Prioritizes real-time temporal freshness (latest frame) over frame completeness.

---

### Q26. How do you measure and analyze microsecond-level performance overheads in StreamOn?

**Category:** Performance & Telemetry  
**Difficulty:** Intermediate    **Priority:** 🟢 Low

**Why the interviewer asks this:**  
To test your knowledge of profiling techniques, micro-benchmarking, and `std::chrono` high-resolution timers in C++.

**Also asked as:**
* How did you quantify the CPU time spent encrypting and decrypting frames?
* What timing metrics are logged in `Server/Logic.cpp`?

**Expected Answer:**  
In [Server/Logic.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/Server/Logic.cpp#L104-L116) and [Client/Logic.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/Client/Logic.cpp#L85-L97), execution latency for frame encryption/decryption is profiled using `std::chrono::high_resolution_clock`:

```cpp
auto start_time = std::chrono::high_resolution_clock::now();

// Execute core crypto loop
for (size_t i = 0; i < buf.size(); i++) buf[i] ^= shared_key[i % shared_key.size()]; 

auto end_time = std::chrono::high_resolution_clock::now();
long long duration = std::chrono::duration_cast<std::chrono::microseconds>(end_time - start_time).count();

static long long total_encryption_time = 0;
static long long encryption_frames = 0;
total_encryption_time += duration;
encryption_frames++;

if (encryption_frames % 30 == 0) { // Log rolling average every 30 frames
    std::cout << "[Server] Avg Encryption Time: " << (total_encryption_time / encryption_frames) << " us\n";
}
```

**Key Profiling Insights:**
* Measures time in **microseconds ($\mu s$)**.
* Uses a rolling counter (`encryption_frames % 30 == 0`) to prevent console I/O flooding from slowing down the main execution loop.

**Important points I must mention:**
* Use of `std::chrono::high_resolution_clock`.
* Microsecond conversion (`std::chrono::microseconds`).
* Throttling log output to every 30 frames to eliminate terminal I/O latency skew.

---

### Q27. Where do buffer copies occur in StreamOn's media pipeline, and how could zero-copy techniques be applied?

**Category:** Performance Optimization  
**Difficulty:** Intermediate    **Priority:** 🟢 Low

**Why the interviewer asks this:**  
High-performance video streaming relies on eliminating unnecessary memory copies (memcpy/vector copies).

**Also asked as:**
* Is memory allocated dynamically on every frame in StreamOn?
* How can you optimize `std::vector<uint8_t>` usage to achieve zero-copy streaming?

**Expected Answer:**  
Currently, StreamOn incurs multiple buffer copy operations per frame:
1. `cv::imencode` writes into a temporary `std::vector<uchar>`.
2. `FrameEncryptor::EncryptFrame` copies memory into chunk vectors, and appends them into a new `encrypted` vector.
3. `StreamServer` copies `encrypted` into `m_latestEncryptedFrame`.
4. `cpp-httplib` copies vector bytes into its HTTP internal response buffer.

**Zero-Copy Optimization Strategies:**
1. **Reusable Ring Buffer Pools:** Pre-allocate a fixed ring buffer pool of byte arrays at startup. Instead of creating new `std::vector` instances on every frame, re-use pre-allocated buffer slots.
2. **Move Semantics (`std::move`):** Transfer ownership of byte buffers using R-value references (`std::move`) rather than l-value copy assignment.
3. **`std::string_view` / `std::span` (C++20):** Pass non-owning pointer/length views (`std::span<const uint8_t>`) down the encryption pipeline instead of passing full `std::vector` objects by value or const-reference.

**Important points I must mention:**
* Current bottleneck: Multiple vector allocations and copies per frame.
* Fixes: Buffer reuse pools, move semantics (`std::move`), and non-owning memory views (`std::span` / pointers).

---

### Q28. How would you scale StreamOn to support 100+ concurrent viewing clients?

**Category:** Scalability & System Design  
**Difficulty:** Advanced    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To test whether you can scale a desktop point-to-point application into a distributed production service.

**Also asked as:**
* If 1,000 users want to watch the live camera stream simultaneously, how would the architecture change?
* What happens to CPU usage if 50 clients poll `GET /frames` at the same time?

**Expected Answer:**  
Currently, `StreamServer` encrypts and serves frames directly to the client. If 1,000 clients poll `StreamServer` simultaneously, the server's network link and CPU will saturate immediately.

**Distributed Scaling Architecture (Publisher-Subscriber Model):**

```
 [StreamOn Camera] ---> Encrypts Frame Once (AES-GCM)
                            |
                            v
             [Media Server Origin Node (NGINX / SRS / Kinesis)]
                            |
             +--------------+--------------+
             |                             |
     [CDN Edge Node 1]             [CDN Edge Node 2]
             |                             |
     (100 Clients)                 (900 Clients)
```

1. **Decouple Capture from Distribution:** The local C++ application acts strictly as an **Ingest Publisher**. It captures, compresses, and encrypts the frame **once**.
2. **Push to Media Origin Server:** The encrypted stream is pushed via WebRTC or RTMP to a Media Gateway / CDN Origin (e.g., NGINX Media Server, AWS Kinesis Video Streams).
3. **CDN Fan-Out:** Edge servers handle the high bandwidth and connection fan-out to 1,000+ viewing clients. The origin server encrypts each frame **only once**, offloading distribution entirely to edge nodes.

**Important points I must mention:**
* Point-to-point architecture does not scale beyond a few clients.
* Frame encryption must happen **once** at the publisher, not once-per-client.
* Use a Media Gateway / CDN (WebRTC / HLS / RTMP) to handle client fan-out.

---

### Q29. What is the primary performance bottleneck of StreamOn, and how would you profile it?

**Category:** Performance & Bottlenecks  
**Difficulty:** Advanced    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To verify your ability to diagnose performance bottlenecks systematically using metrics rather than guessing.

**Also asked as:**
* What limits the maximum FPS of StreamOn? Is it CPU, RAM, Network, or Disk I/O?
* How would you use Visual Studio Profiler or `perf` to locate bottlenecks in this application?

**Expected Answer:**  
The primary performance bottlenecks in StreamOn differ by build:

1. **HTTPS GUI Build Bottleneck (CPU Bound):**
   * *Primary Bottleneck:* **RSA Modular Exponentiation** inside OpenSSL. Slicing frames into 214-byte blocks and executing 100+ RSA operations per frame saturates CPU cores.
   * *Secondary Bottleneck:* Network latency introduced by client-side HTTP polling loops (`sleep_for(33ms)`).
2. **UDP Build Bottleneck (Network / Packet Loss Bound):**
   * *Primary Bottleneck:* Lack of congestion control. Sending unthrottled UDP datagram bursts causes socket buffer drops (`SO_SNDBUF` / `SO_RCVBUF`).

**Profiling Methodology:**
Run the Visual Studio **Performance Profiler (ALT + F2)** using CPU Usage sampling:
* Inspect the call tree. `RSACrypto::EncryptBlock` $\rightarrow$ `EVP_PKEY_encrypt` will show > 80% CPU sample time.
* Monitor thread contention around `m_frameMutex` using the Concurrency Visualizer.

**Important points I must mention:**
* Bottleneck classification: HTTPS build is **CPU-bound** (RSA math); UDP build is **Socket Buffer / Network bound**.
* Profiling tools: Visual Studio CPU Profiler, Concurrency Visualizer.

---

### Q30. How does the UDP build handle packet loss, chunk dropping, and out-of-order packet reassembly?

**Category:** Reliability & Network Protocols  
**Difficulty:** Intermediate - Advanced    **Priority:** 🔴 High

**Why the interviewer asks this:**  
UDP does not guarantee delivery or packet ordering. Interviewers want to see how your code reassembles fragmented frames robustly.

**Also asked as:**
* What happens in `Client/Logic.cpp` if packet #2 of 5 is lost during UDP streaming?
* How do you reconstruct a frame from out-of-order UDP chunks?

**Expected Answer:**  
In [Client/Logic.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/Client/Logic.cpp#L59-L113), the client maintains an internal reassembly state for incoming UDP packets:

```cpp
static uint32_t currentFrameId = 0;
static vector<vector<uint8_t>> frameChunks;
static uint16_t chunksReceived = 0;

// On receiving packet:
if (frameId > currentFrameId) {
    currentFrameId = frameId;
    frameChunks.assign(totalChunks, vector<uint8_t>()); // Re-allocate slots for new frame
    chunksReceived = 0;
}

if (frameId == currentFrameId && chunkIdx < totalChunks) {
    if (frameChunks[chunkIdx].empty()) { 
        frameChunks[chunkIdx] = payload; // Slot chunk at exact index
        chunksReceived++;
        if (chunksReceived == totalChunks) { 
            // All chunks received! Reassemble & Decode JPEG
        }
    }
}
```

**Failure Scenarios & Current Limitations:**
* **Dropped Chunk Scenario:** If 1 chunk out of 10 is lost on the network, `chunksReceived` will equal 9, never matching `totalChunks`. When a newer `frameId` arrives, the incomplete frame is silently abandoned and overwritten.
* **Out-of-Order Delivery:** Out-of-order chunks within the *same* frame are handled correctly because chunks are inserted by array index (`frameChunks[chunkIdx]`). However, if a chunk from `frameId = 5` arrives *after* `frameId = 6` has started, it is discarded.

**Important points I must mention:**
* Frame slotting using `chunkIdx` handles out-of-order arrival *within* the current frame.
* Incomplete frames are dropped as soon as a higher `frameId` arrives.
* Trade-off: Prevents display of corrupted JPEG buffers at the expense of dropped frames.

---

### Q31. What happens if the network disconnects or invalid data is received during key exchange or streaming?

**Category:** Reliability & Fault Tolerance  
**Difficulty:** Intermediate    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To test exception handling, error boundary protection, and crash resilience.

**Also asked as:**
* How does StreamOn recover if the server crashes or the network drops during the handshake?
* Will malformed data cause a segfault or memory access violation?

**Expected Answer:**  
StreamOn enforces defensive error boundaries across networking and crypto modules:

1. **Handshake Failure Handling:**
   If `StreamClient::PerformHandshake()` fails (e.g., HTTP 500 error or corrupted JSON), `Connect()` returns `false`, logs the error, frees `m_client`, and leaves `m_handshakeDone` set to `false`. No video streaming threads are spawned.
2. **Corrupted Network Payloads:**
   If corrupted bytes are received during frame polling:
   * `RSACrypto::DecryptBlock` fails OpenSSL's OAEP padding validation and returns an empty vector `{}`.
   * `FrameDecryptor::DecryptFrame` detects the empty return, logs a warning (`Decryption failed`), and aborts frame reassembly without crashing.
   * `cv::imdecode` handles invalid byte arrays by returning an empty `cv::Mat`, which is caught by `if (!frame.empty())` checks before rendering.
3. **Socket Error Recovery ([Client/Logic.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/Client/Logic.cpp#L16)):**
   WinSock errors during socket binding or frame receipt trigger status updates (`appStatus = "Fatal: Socket BIND ERROR"`) without causing memory access violations.

**Important points I must mention:**
* Fail-fast checks at every step: JSON parsing $\rightarrow$ RSA decryption $\rightarrow$ JPEG decode.
* Defensive `empty()` checks prevent null pointer dereferences or GDI crashes on invalid frames.

---

### Q32. How would you design a complete unit testing and integration testing strategy for StreamOn?

**Category:** Testing & Quality Assurance  
**Difficulty:** Intermediate    **Priority:** 🔴 High

**Why the interviewer asks this:**  
To test whether you know how to write testable C++ code using frameworks like GoogleTest (gtest).

**Also asked as:**
* How do you test cryptographic components like `RSACrypto` and `KyberKEM` without running the full GUI?
* What edge cases should be tested in `FrameEncryptor`?

**Expected Answer:**  
A robust C++ testing strategy for StreamOn requires GoogleTest (gtest) across three tiers:

1. **Unit Tests (Cryptography & Utilities):**
   * *KyberKEM Test:* Verify that `Encapsulate(pubKey)` and `Decapsulate(ciphertext)` yield byte-for-byte identical 32-byte shared secrets.
   * *RSACrypto Test:* Test round-trip encryption/decryption of a 214-byte payload. Verify that attempting to encrypt 215 bytes throws an expected error.
   * *HEX/Byte Utils Test:* Test `BytesToHex` and `HexToBytes` accuracy.
2. **Integration Tests (Frame Pipeline):**
   * Feed a dummy 640x480 synthetic `cv::Mat` frame into `FrameEncryptor`, pass output to `FrameDecryptor`, and verify that `cv::imdecode` reconstructs an identical image matrix.
3. **Network Mock Tests:**
   * Mock `cpp-httplib` routes to return simulated HTTP `200 OK` responses containing corrupted JSON payloads to test client error recovery.

**Important points I must mention:**
* Unit testing low-level components (`KyberKEM`, `RSACrypto`) independently using GoogleTest.
* Round-trip testing (Plaintext $\rightarrow$ Encrypt $\rightarrow$ Decrypt $\rightarrow$ Verify Equality).
* Synthetic frame generation for headless CI/CD pipeline integration.

---

### Q33. How would you debug frame corruption, visual tearing, or OpenSSL error stacks in StreamOn?

**Category:** Debugging & Troubleshooting  
**Difficulty:** Intermediate    **Priority:** 🟡 Medium

**Why the interviewer asks this:**  
To test your practical debugging skills when diagnosing real-world C++ issues.

**Also asked as:**
* How does `LogOpenSSLError` work in `RSACrypto.cpp`?
* If the video output is showing garbage pixels or tearing, how would you isolate the bug?

**Expected Answer:**  
StreamOn provides dedicated logging hooks for debugging crypto and rendering bugs:

1. **OpenSSL Error Stack Inspection ([RSACrypto.cpp](file:///d:/Mini_Project/StreamOn/StreamOn/UI/Server-client_UI/RSACrypto.cpp#L511-L516)):**
   OpenSSL errors are pushed onto an internal thread-local error queue. `RSACrypto::LogOpenSSLError` pops error codes from the stack and converts them into human-readable strings:
```cpp
void RSACrypto::LogOpenSSLError(const std::string& context) const {
    unsigned long err = ERR_get_error(); // Pop error code from OpenSSL queue
    char buf[256];
    ERR_error_string_n(err, buf, sizeof(buf)); // Format error message
    LogError(context + " -> " + std::string(buf));
}
```
2. **Diagnosing Visual Tearing / Image Corruption:**
   * *Step 1 (Isolate Network):* Save raw JPEG bytes to disk on the server *before* encryption and on the client *after* decryption. Run a binary `diff`.
   * *Step 2 (Isolate Decryption):* If binary outputs differ, print the hex hash (`BytesToHex`) of each encrypted chunk to verify if network packets dropped or corrupted bytes in transit.
   * *Step 3 (Isolate GDI):* If decrypted JPEG bytes match, the issue lies in GDI color space conversion (`BGR` vs `BGRA`) or `BITMAPINFO` pitch/stride alignment inside `ServerWindow.cpp`.

**Important points I must mention:**
* Use of `ERR_get_error()` and `ERR_error_string_n()` to inspect OpenSSL's internal error stack.
* Isolation strategy: Save binary JPEG files before encryption and after decryption to isolate network corruption from GDI rendering bugs.

---

### Q34. Explain the build system setup in StreamOn. How does `build_all.bat` configure environment variables and link dependencies?

**Category:** Deployment & Build System  
**Difficulty:** Basic - Intermediate    **Priority:** 🟢 Low

**Why the interviewer asks this:**  
To evaluate your build configuration skills, C++ compiler options (MSVC), and third-party library linking on Windows.

**Also asked as:**
* How do you build StreamOn from the command line?
* How are OpenCV, OpenSSL, and liboqs linked in Visual Studio / CMake?

**Expected Answer:**  
StreamOn uses two build mechanisms:
1. **Batch Build Script ([build_all.bat](file:///d:/Mini_Project/StreamOn/StreamOn/build_all.bat)):** Command-line script using Microsoft Visual C++ Compiler (`cl.exe`).
2. **CMake Build System (`UI/Server-client_UI/CMakeLists.txt`):** IDE-agnostic build setup for Visual Studio 2022.

**Key Environment Variables & Linking in `build_all.bat`:**
* `STREAMON_OQS_DIR`: Points to `liboqs` headers and static/dynamic library (`oqs.lib` / `oqs.dll`).
* `STREAMON_OPENCV_DIR`: Points to OpenCV include headers and import library (`opencv_world4120.lib`).
* **WinSock & System Libraries:** Explicitly links Windows socket libraries (`ws2_32.lib`) and Common Controls (`comctl32.lib`):

```bat
:: Compiler Flags in build_all.bat
cl.exe /std:c++17 /EHsc /I"%STREAMON_OQS_DIR%\include" /I"%STREAMON_OPENCV_DIR%\include" ... main.cpp /link /LIBPATH:"..." opencv_world4120.lib oqs.lib ws2_32.lib comctl32.lib
```

**DLL Deployment:** After compilation, `build_all.bat` automatically copies runtime dynamic link libraries (`opencv_world4120.dll`, `oqs.dll`) into the executable directory so binaries launch without `DLL Not Found` errors.

**Important points I must mention:**
* Use of C++17 ISO standard flag (`/std:c++17`).
* Explicit linkage against `ws2_32.lib` (WinSock2) and `comctl32.lib` (Win32 GUI).
* Post-build step copying required DLLs (`oqs.dll`, `opencv_world4120.dll`) to binary output paths.

---

### Q35. If you were to rebuild StreamOn from scratch for production (StreamOn 2.0), what top 5 architectural redesigns would you make?

**Category:** Improvement & Production Readiness  
**Difficulty:** Advanced    **Priority:** 🔴 High

**Why the interviewer asks this:**  
This is the ultimate architectural maturity check. It tests whether you can critically evaluate your own prototype and articulate a clear engineering roadmap for production deployment.

**Also asked as:**
* What are the biggest limitations of StreamOn today, and how would you fix them?
* How would you turn this prototype into a production-grade secure streaming product?

**Expected Answer:**  
If rebuilding StreamOn for enterprise production (StreamOn 2.0), I would implement 5 key architectural changes:

1. **Replace RSA Frame Encryption with AES-256-GCM / ChaCha20-Poly1305:**
   Use Kyber512 during the handshake to derive a 256-bit symmetric session key. Encrypt live video payloads using **AES-256-GCM**, leveraging hardware **AES-NI** CPU instructions. This eliminates the RSA CPU bottleneck and reduces encryption time from milliseconds to microseconds.
2. **Replace HTTP GET Polling with WebSockets / WebRTC (SRTP):**
   Eliminate client polling loops (`GET /frames`). Implement full-duplex WebSockets over TLS or **SRTP (Secure Real-Time Transport Protocol)** over UDP for ultra-low latency push streaming.
3. **Add Authenticated Encryption (AEAD Integrity Checks):**
   Attach an authentication tag (HMAC or GCM tag) and frame counter to every packet to detect payload tampering and prevent replay attacks.
4. **Implement a Zero-Copy Ring Buffer Memory Pool:**
   Pre-allocate fixed image buffer pools at startup, using `std::span` and move semantics to eliminate intermediate `std::vector` allocations and memory copies.
5. **Cross-Platform Abstraction (Linux / macOS Support):**
   Replace Windows-specific Win32 GDI rendering and WinSock code with cross-platform abstractions (GLFW/Qt for rendering, ASIO/uvw for asynchronous networking).

**Important points I must mention:**
* Transition to AES-256-GCM for stream payloads.
* Transition to WebSockets / WebRTC for push transport.
* AEAD tags for integrity/anti-tampering.
* Zero-copy buffer pools for memory optimization.
* Cross-platform abstraction.

---

## 🔴 1. Must-Know Questions

If time is limited in an interview, you **must** master these 7 core questions first:

1. **Q1:** Project overview, problem statement, and core objectives.
2. **Q2:** Why Post-Quantum Cryptography (Kyber512) is necessary (SNDL attacks).
3. **Q3:** End-to-end request/response workflow from camera capture to screen display.
4. **Q4:** Complete 2-step cryptographic handshake sequence (`GET /keys`, `POST /handshake`).
5. **Q9:** RSA-2048 block chunk size limit math ($256 - 42 = 214 \text{ bytes}$).
6. **Q10:** Why Kyber512 (KEM) and RSA are combined in a hybrid model.
7. **Q23:** Why RSA frame encryption is an architectural trade-off/anti-pattern and how AES-GCM fixes it.

---

## ⚠️ 2. Project Weak Points & Vulnerabilities

Be prepared to defend these 4 project vulnerabilities if an interviewer probes your architecture:

1. **RSA Payload Encryption Bottleneck:** RSA modular exponentiation is CPU-intensive. Slicing JPEG frames into 214-byte blocks forces hundreds of RSA operations per frame, bottlenecking FPS. *(Defense: Acknowledge it as a prototype implementation and explain the AES-256-GCM hybrid roadmap).*
2. **UDP XOR Cipher Vulnerability:** In `Server/Logic.cpp`, XORing frames with a static key repeats every 32 bytes and is vulnerable to Known-Plaintext Attacks via standard JPEG headers (`0xFF 0xD8`). *(Defense: Explain that this was built for microsecond testing and must be replaced by ChaCha20-Poly1305).*
3. **HTTP Polling Latency:** Client-side HTTP polling (`GET /frames`) with `sleep_for(33ms)` introduces latency and HTTP header overhead. *(Defense: Propose WebSockets or SRTP over UDP for push-based streaming).*
4. **Disabled Certificate Verification:** `StreamClient` sets `enable_server_certificate_verification(false)` to bypass self-signed cert checks. *(Defense: Note that local testing required self-signed certs, but production requires CA chain verification and certificate pinning).*

---

## 💣 3. High-Risk Follow-Ups

These follow-up questions can catch candidates who have shallow understanding:

* **Interviewer:** *"If Kyber generates a 32-byte shared secret during the handshake, why are you still encrypting frames with RSA keys in `FrameEncryptor` instead of using that shared secret?"*  
  **Your Answer:** In the GUI build, the Kyber shared secret is logged and derived as a session identifier, while frame chunks are encrypted using the remote RSA public key to demonstrate asymmetric OAEP padding. In a production pipeline, that 32-byte Kyber shared secret would directly seed an AES-256-GCM symmetric key for payload encryption.
* **Interviewer:** *"Why did you set negative height (`-bgra.rows`) in the `BITMAPINFOHEADER` during GDI rendering?"*  
  **Your Answer:** In Windows GDI, a positive `biHeight` indicates a bottom-up DIB (where row 0 is the bottom of the image), which displays OpenCV frames upside down. Setting `biHeight` to a negative value specifies a top-down DIB, displaying the frame right-side up.

---

## 📚 4. Core Concepts to Revise Before Interview

Review these fundamental Computer Science topics before your interview:

* **Cryptography:** Asymmetric vs Symmetric Ciphers, KEM (Key Encapsulation Mechanisms), RSA-2048 OAEP Padding Math, Lattice-based Cryptography (LWE/Kyber), AEAD (Authenticated Encryption with Associated Data).
* **C++ Systems Programming:** C++17 Standards, RAII & Resource Cleanup (`OQS_MEM_cleanse`), OpenSSL C API (`EVP_PKEY`, `BIO`), `std::thread`, `std::mutex`, `std::atomic`.
* **Networking & Media:** TCP vs UDP, Application-level MTU Fragmentation, HTTP Polling vs WebSockets vs RTP, JPEG Compression (`cv::imencode`).
* **Windows Desktop API:** Win32 Message Loops (`GetMessage`/`PostMessage`), GDI Bitmap Rendering (`SetDIBitsToDevice`), DirectShow capture backends.

---

## ✅ 5. Interview Readiness Checklist

Use this checklist for your final self-assessment:

- [ ] Can explain project in 30 seconds
- [ ] Can explain project in 1 minute
- [ ] Can explain dual architecture (HTTPS REST vs UDP sockets)
- [ ] Can explain complete 5-step end-to-end workflow
- [ ] Can defend hybrid PQC technology choices (Kyber512 + RSA)
- [ ] Can explain important files (`KeyManager`, `RSACrypto`, `KyberKEM`, `FrameEncryptor`)
- [ ] Can explain core implementation details (OpenSSL BIOs, `liboqs` C API)
- [ ] Can defend design decisions & explain RSA chunking math ($256 - 42 = 214$)
- [ ] Can discuss CPU/bandwidth bottlenecks and microsecond timing metrics
- [ ] Can explain UDP MTU chunk fragmentation (9-byte custom binary header)
- [ ] Can discuss error handling, corrupted frame dropping, and memory sanitization
- [ ] Can explain TLS certificate handling and security implications
- [ ] Can explain CMake & Visual Studio batch build setup
- [ ] Can propose 5 concrete engineering improvements for StreamOn 2.0
- [ ] Can defend every claim on your resume regarding this project