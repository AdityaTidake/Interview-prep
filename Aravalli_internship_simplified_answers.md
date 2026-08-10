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


### 📁 Files referred to during the project analysis

The repository analysis referred to these main files: `README.md`, `Protocol.h`, `RSACrypto.h`, `RSACrypto.cpp`, `kyberKEM.h`, `kyberKEM.cpp`, `keyManager.h`, `keyManager.cpp`, `FrameEncryptor.h`, `FrameEncrypter.cpp`, `FrameDecryptor.h`, `FrameDecryptor.cpp`, `StreamServer.h`, `StreamClient.h`, `StreamClient.cpp`, `ServerWindow.cpp`, `main.cpp`, `Server/Logic.cpp`, `Client/Logic.cpp`, and the UI/build files under `UI/Server-client_UI/` (including `CMakeLists.txt` and `build_all.bat`).

The explanations below are intentionally **interviewer-friendly**: they keep the original technical idea but avoid unnecessary file names, code paths, and implementation details.

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
StreamOn is a C++ desktop prototype for secure real-time webcam streaming. Its main goal is to protect live video from interception and also consider future quantum-computing threats. It has two versions: an HTTPS-based version with a Win32 GUI and a low-latency UDP version. The security design combines Kyber512 for post-quantum key exchange with RSA-2048 in the current prototype.


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
Post-Quantum Cryptography means cryptography designed to remain secure even against powerful quantum computers. RSA and ECC could eventually be broken by Shor's Algorithm, so encrypted traffic captured today could potentially be decrypted later. Kyber512 is a lattice-based KEM designed to resist such attacks, so StreamOn uses it to establish a session secret with quantum resistance in mind.


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
A simple way to explain the flow is: **camera → JPEG compression → encryption → network → decryption → JPEG decoding → display**. The camera captures a frame, the frame is compressed to reduce its size, the current HTTPS version encrypts it in RSA-sized chunks, and the server makes the latest encrypted frame available. The client downloads it, decrypts it, converts it back into an image, and displays it.


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
Before streaming, the client and server establish a shared secret. First, the client gets the server's public keys. Then the client sends its public keys, and the server uses the client's Kyber public key to create a Kyber ciphertext and a shared secret. The client decapsulates that ciphertext with its private Kyber key, so both sides end up with the same 32-byte shared secret. The handshake uses `GET /keys` and `POST /handshake`.


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
StreamOn is organized into separate layers so each part has one main responsibility: camera/media capture, cryptography, frame processing, networking, and the user interface. This separation makes the code easier to understand and change. `KeyManager` acts as a higher-level coordinator for the RSA and Kyber components, while callbacks and buffers keep the layers loosely connected.


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
The application separates work across the UI thread, camera thread, and network threads. The UI thread handles Windows messages and should not perform slow work. Camera and network operations run in the background. Mutexes protect shared frame data, atomics handle simple status flags, and `PostMessage` tells the UI thread when a new frame is ready. This prevents race conditions and keeps the interface responsive.


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
The project has two network approaches because they demonstrate different trade-offs. **HTTPS/TCP polling** is easier to integrate with a GUI and provides reliable, structured communication, but it adds polling and TCP latency. **UDP** is faster and better suited to real-time video because it avoids TCP head-of-line blocking, but packets can be lost or arrive out of order. For real-time streaming, UDP is generally the better starting point.


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
Kyber512 is integrated through `liboqs`. The basic process is simple: create a Kyber object, generate a public/private key pair, use the public key to encapsulate a secret, and use the private key to decapsulate it. For Kyber512, the important sizes are 800 bytes for the public key, 1632 bytes for the private key, 768 bytes for the ciphertext, and 32 bytes for the shared secret.


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
RSA-2048 works with 256-byte ciphertext blocks because 2048 bits equals 256 bytes. With OAEP using SHA-1, 42 bytes are used for padding, leaving **256 − 42 = 214 bytes** for actual plaintext. That is why a JPEG frame is split into chunks of at most 214 bytes before RSA encryption. Each encrypted chunk becomes exactly 256 bytes.


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
Kyber and RSA have different roles. Kyber is a **KEM**, so its job is to establish a shared secret; it is not designed to encrypt a large video stream directly. RSA is used in the current prototype for asymmetric operations and frame encryption. The hybrid idea gives the project a post-quantum key-establishment component while retaining the classical RSA component. In a production design, the shared secret would be used with a fast symmetric cipher such as AES-256-GCM.


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
In the UDP prototype, each byte of the JPEG is XORed with a repeating 32-byte shared key. This is very fast, but it is **not secure encryption**. JPEG files have predictable header bytes, so an attacker can use known plaintext to recover parts of the repeating key. Reusing the same key without a nonce also creates additional weaknesses. A production version should replace this with AES-256-GCM or ChaCha20-Poly1305.


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
RSA keys are stored in PEM format, while Kyber keys are stored as binary data. The application also tries to protect private key material in memory by securely clearing sensitive Kyber buffers before releasing them. The important idea is that deleting a C++ vector does not necessarily erase the old bytes from memory, so sensitive key material should be explicitly cleansed.


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
The HTTPS version uses TLS so the connection between client and server is encrypted. The server provides a certificate and private key, while the client creates an SSL connection. Certificate verification is disabled in the current local setup so self-signed certificates can be used easily. That is acceptable for a prototype, but production systems must verify the certificate chain, and certificate pinning can be added when appropriate to reduce MITM risk.


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
The camera is captured through OpenCV. A raw 640×480 BGR frame is roughly 920 KB, so sending it directly would require too much bandwidth. StreamOn compresses the frame to JPEG before transmission. The JPEG quality is set to 45, which greatly reduces the frame size while keeping reasonable visual quality. The frame rate is controlled by the capture loop and timing.


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
UDP packets cannot be made arbitrarily large because large packets may be fragmented by the network. StreamOn therefore keeps the encrypted payload around 1200 bytes per packet, safely below the usual Ethernet MTU limit. Each packet also has a small header containing the packet type, frame ID, total number of chunks, and current chunk index. The receiver uses these fields to rebuild the original frame.


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
In the GUI version, OpenCV frames are converted from BGR to a format suitable for Windows GDI and then drawn directly inside a Win32 window. GDI was chosen because it allows the video to be embedded into the application's existing `HWND` controls instead of opening a separate OpenCV window. A negative bitmap height is used so the image is treated as top-down and is displayed in the correct orientation.


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
When recording is enabled, StreamOn saves the video as an AVI file using the MJPEG codec at about 20 FPS. The same captured frames can be sent to the streaming pipeline and the recorder. The main limitation is that synchronous disk writing can add latency, so a production version should move recording to a background worker with a queue.


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
`KeyManager` is the component that coordinates the application's RSA and Kyber keys. It keeps track of local private/public keys, the remote public keys, whether local keys exist, and whether the handshake has completed. Its main responsibilities are generating or loading keys, processing the server side of the handshake, and processing the client side of the handshake. In simple terms, it manages the application's key lifecycle.


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
OpenSSL uses C-style objects that must be released manually. StreamOn uses memory BIOs when it needs to convert OpenSSL key data into strings such as PEM. After the data is copied into a C++ string, the BIO is freed. RSA key objects are also released when their owning C++ object is destroyed. The goal is to follow RAII-style ownership and avoid memory leaks.


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
`FrameEncryptor` takes a compressed JPEG frame and breaks it into chunks small enough for RSA-OAEP. Each chunk is encrypted separately and the encrypted blocks are joined into one output buffer. It checks that the RSA key is available and that the frame is not empty, and it stops if encryption fails. Pre-allocating the output buffer also reduces unnecessary memory reallocations.


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
Before decrypting, `FrameDecryptor` checks that the ciphertext size is a multiple of the RSA block size, which is 256 bytes. It then decrypts each block and joins the plaintext chunks back together. If any block fails decryption, the whole frame is rejected. The resulting bytes should form the original JPEG, which can then be decoded and displayed.


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
The HTTPS version uses `cpp-httplib` with OpenSSL support. The main endpoints are simple: `GET /keys` provides public keys, `POST /handshake` performs the Kyber key exchange, and `GET /frames` returns the latest encrypted video frame. The HTTPS server runs in a background thread so it does not block the GUI.


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
Using RSA to encrypt every video frame is one of the biggest weaknesses of the current prototype. RSA is computationally expensive and can only encrypt small chunks, so a single 30 KB frame may require many RSA operations. It also increases the amount of data sent. The correct production approach is to use RSA/Kyber only during the handshake and then use a fast symmetric cipher such as AES-256-GCM or ChaCha20-Poly1305 for the video payload.


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
HTTP polling makes the client repeatedly ask for a new frame, which creates unnecessary request overhead and adds latency. WebSockets provide a persistent connection and allow the server to push data more efficiently. RTP/RTSP over UDP is designed more specifically for real-time media and can provide lower latency. For a production real-time system, WebRTC or RTP/SRTP would generally be more suitable than repeated HTTP polling.


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
StreamOn keeps only the latest encrypted frame instead of storing an unlimited queue of frames. When a new frame arrives, it replaces the previous one. This keeps memory usage bounded and ensures that a slow client receives a recent frame rather than a very old one. The trade-off is that intermediate frames are dropped when the network cannot keep up.


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
The UDP implementation measures encryption and decryption time with `std::chrono::high_resolution_clock` and records the result in microseconds. It calculates an average over multiple frames instead of printing every measurement. This gives a simple way to see whether the cryptographic operation is becoming a performance bottleneck without adding too much logging overhead.


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
There are several places where frame data can be copied: during JPEG encoding, encryption chunk creation, frame buffering, and network response handling. These repeated allocations and copies can hurt performance. A better design would reuse pre-allocated buffers, use move semantics where ownership can be transferred, and pass non-owning views such as `std::span` where appropriate. The goal is to reduce unnecessary memory movement.


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
The current point-to-point design is not ideal for 100+ viewers because the server would have to handle many connections and a large amount of outgoing traffic. I would separate **capture/ingest** from **distribution**. The C++ application would capture and encode the stream once, then send it to a media server or streaming gateway. That system could then distribute the stream to many clients using WebRTC, HLS, or another suitable delivery system. This moves the scaling problem away from the desktop application.


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
The main bottleneck depends on the build. In the HTTPS version, RSA frame encryption is the major CPU cost, while HTTP polling also adds latency. In the UDP version, network and socket-buffer pressure can become the main limitation. I would confirm this with a profiler rather than guessing—for example, using Visual Studio's CPU and concurrency profilers to see where CPU time and thread contention are actually occurring.


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
UDP does not guarantee delivery or order. StreamOn uses the frame ID and chunk index to place each received chunk in the correct position. Therefore, chunks that arrive out of order can still be reconstructed correctly. If even one chunk is lost, the frame cannot be completed, so the incomplete frame is discarded when a newer frame arrives. This favors real-time playback over perfect delivery.


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
The application uses defensive checks at each stage. If the handshake fails, streaming does not start. If received data is malformed, decryption can fail and the frame is dropped instead of being displayed. Invalid JPEG data is also rejected before rendering. The general approach is to fail safely at each boundary—networking, key exchange, decryption, and image decoding—rather than allowing bad input to crash the application.


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
I would test StreamOn at three levels. **Unit tests** would cover RSA operations, Kyber operations, encryption/decryption, chunking, key handling, and packet parsing. **Integration tests** would verify the complete handshake and end-to-end frame flow between a client and server. **Failure tests** would intentionally use corrupted data, missing packets, wrong keys, disconnects, and invalid frames. For video, I would also verify that decrypted frames match the original frames and measure performance such as FPS and encryption time.


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
I would debug the problem by finding which stage first becomes incorrect. First, compare the JPEG before encryption with the JPEG after decryption. If they differ, investigate encryption, decryption, or network transmission. If the bytes match but the image looks wrong, investigate JPEG decoding and the Windows rendering path. For OpenSSL failures, inspect the OpenSSL error stack to find the actual cryptographic error. This isolates the problem instead of debugging the whole pipeline at once.


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
StreamOn can be built using the Windows C++ toolchain and its configured dependencies. The build configuration provides paths to libraries such as OpenCV and liboqs and links the required Windows networking and GUI libraries. The project uses C++17, and the required runtime DLLs are placed beside the executable so the application can start correctly. The main idea is that the build system prepares the compiler, include paths, libraries, and runtime dependencies.


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
For a production StreamOn 2.0, my top five changes would be: **1)** use Kyber/RSA only for the handshake and AES-256-GCM or ChaCha20-Poly1305 for video encryption; **2)** replace HTTP polling with WebRTC, WebSockets, or RTP/SRTP; **3)** add authenticated encryption, nonces, frame counters, and replay protection; **4)** use reusable buffer pools to reduce copies and allocations; and **5)** remove Windows-specific dependencies where possible so the networking and UI layers can be made cross-platform. These changes would improve security, latency, performance, scalability, and maintainability.


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