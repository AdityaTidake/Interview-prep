# StreamOn 🔒 — Important Terms & Definitions

## 📑 Index

1. [Project & Software Concepts](#1-project--software-concepts)
2. [Networking Terms](#2-networking-terms)
3. [Cryptography Terms](#3-cryptography-terms)
4. [Post-Quantum Cryptography Terms](#4-post-quantum-cryptography-terms)
5. [RSA & OpenSSL Terms](#5-rsa--openssl-terms)
6. [Video & Image Processing Terms](#6-video--image-processing-terms)
7. [C++ & Concurrency Terms](#7-c--concurrency-terms)
8. [Windows & UI Terms](#8-windows--ui-terms)
9. [Security Terms & Vulnerabilities](#9-security-terms--vulnerabilities)
10. [Performance & Optimization Terms](#10-performance--optimization-terms)
11. [Protocol & Data Format Terms](#11-protocol--data-format-terms)
12. [Interview Quick-Revision Sheet](#12-interview-quick-revision-sheet)

---

### How to use this glossary

For each term, the **Where it is used in the project** line connects the definition directly to StreamOn. This helps you answer both **“What is it?”** and **“Where did you use it?”** during an interview.

---

# 1. Project & Software Concepts

## StreamOn
**Definition:**  
StreamOn is a C++ desktop application prototype for securely streaming webcam video between a server and a client.

**Simple explanation:**  
It captures video, compresses it, encrypts it, sends it over a network, decrypts it on the client, and displays it.

**Interview line:**  
> "StreamOn is a secure real-time webcam streaming prototype built in C++ with video processing, networking, and hybrid cryptography."

---

## End-to-End Encryption (E2EE)
**Definition:**  
End-to-end encryption means data is encrypted at the sender and decrypted only at the intended receiver.

**Where it is used in the project:** It describes the project's overall goal of protecting the video between the streaming endpoints.
**Simple explanation:**  
Someone in the middle of the network should not be able to understand the video.

**Important:**  
Encryption protects **confidentiality**. Authentication and integrity are separate security properties.

---
## Client-Server Architecture
**Definition:**  
A software architecture where one side provides a service (server) and another side consumes it (client).

```text
Client
  │
  │ Requests / receives video
  ↓
Server
  │
  │ Captures and processes video
  ↓
Camera
```

---

## Layered Architecture
**Definition:**  
A design where responsibilities are divided into separate layers.

```text
UI / Presentation
       ↓
Networking
       ↓
Payload Processing
       ↓
Cryptography
       ↓
Camera / Media
```

**Where it is used in the project:** It is reflected in the separation between UI, networking, payload processing, cryptography, and camera/media code.
**Why it helps:**  
Each layer has a focused responsibility, making the project easier to understand and maintain.

---
## Separation of Concerns (SoC)
**Definition:**  
Keeping different responsibilities separate instead of putting everything into one class or module.

**Where it is used in the project:** It is applied by keeping camera capture, networking, cryptography, and UI responsibilities separate.
**Example:**  
Camera capture should not be responsible for RSA encryption, and the UI should not directly manage network packets.

---
## Facade Pattern
**Definition:**  
A design pattern that provides a simple interface over several complicated components.

**Where it is used in the project:**  
`KeyManager` acts as a high-level interface over RSA and Kyber operations.

**Simple explanation:**  
Instead of the rest of the program dealing directly with many cryptographic APIs, it can ask `KeyManager` to perform key-related operations.

---

# 2. Networking Terms

## Network Protocol
**Definition:**  
A set of rules that defines how devices communicate.

**Where it is used in the project:** It applies to the HTTP/HTTPS and UDP communication used by the client and server.
**Examples in StreamOn:**  
HTTP/HTTPS and UDP.

---
## HTTP
**Definition:**  
Hypertext Transfer Protocol. It is an application-layer protocol commonly used for request-response communication.

**Where it is used in the project:**  
HTTP-style endpoints are used for key exchange and retrieving frames.

---

## HTTPS
**Definition:**  
HTTP carried over TLS, providing encrypted communication between client and server.

```text
HTTP
 +
TLS
 ↓
HTTPS
```

---

## TCP
**Definition:**  
Transmission Control Protocol is a connection-oriented transport protocol that provides reliable, ordered delivery.

**Advantages:**  
- Reliable delivery
- Ordered data
- Retransmission of lost data

**Disadvantage for live video:**  
A delayed packet can hold up later data, contributing to latency.

---

## UDP
**Definition:**  
User Datagram Protocol is a connectionless transport protocol that sends independent datagrams without guaranteeing delivery or ordering.

**Advantages:**  
- Low overhead
- Low latency
- No TCP-style retransmission delay

**Disadvantages:**  
Packets can be lost, duplicated, or arrive out of order.

---

## Datagram
**Definition:**  
A self-contained packet sent using a protocol such as UDP.

**Simple explanation:**  
Each UDP packet is treated as an independent message.

---

## Polling
**Definition:**  
Polling means repeatedly asking a server whether new data is available.

```text
Client → "Any new frame?"
Server → "Yes/No"
Client → waits
Client → "Any new frame?"
...
```

**Problem:**  
Repeated requests add overhead and can increase latency.

---

## WebSocket
**Definition:**  
A communication protocol that keeps a connection open and allows both sides to send data without repeatedly creating separate HTTP requests.

**Why useful for streaming:**  
The server can push frames as soon as they are ready.

---

## RTP
**Definition:**  
Real-time Transport Protocol is designed for delivering audio and video over networks.

**Why it matters:**  
It provides concepts such as sequence numbers and timestamps that are useful for real-time media.

---

## RTSP
**Definition:**  
Real Time Streaming Protocol is a control protocol commonly used to control streaming sessions.

**Important distinction:**  
RTSP mainly controls the session; RTP is commonly used to carry the actual media.

---

## MTU
**Definition:**  
Maximum Transmission Unit is the largest packet size that can normally be transmitted over a network link without fragmentation at that layer.

**Example:**  
Ethernet commonly uses an MTU of 1500 bytes.

**Where it is used in the project:**  
UDP application chunks are kept smaller than the typical Ethernet limit to reduce fragmentation risk.

---

## Fragmentation
**Definition:**  
Breaking a large piece of data into smaller pieces so it can travel through a network.

**Where it is used in the project:**  
A large encrypted frame is divided into smaller UDP chunks.

---

# 3. Cryptography Terms

## Cryptography
**Definition:**  
**Where it is used in the project:** It is used for key establishment, encryption, decryption, and TLS security in the streaming pipeline.
The science of protecting information using mathematical techniques.

It provides properties such as:

- Confidentiality
- Integrity
- Authentication
- Non-repudiation

---
## Encryption
**Definition:**  
Converting readable data (plaintext) into unreadable data (ciphertext) using a cryptographic algorithm and key.

```text
Plaintext + Key
      ↓
  Encryption
      ↓
Ciphertext
```

---

## Decryption
**Definition:**  
Converting ciphertext back into the original plaintext using the appropriate key.

---

## Plaintext
**Definition:**  
The original readable data before encryption.

**Where it is used in the project:** The compressed JPEG frame is treated as plaintext immediately before the payload-encryption stage.
**Example:**  
A JPEG frame before encryption.

---
## Ciphertext
**Definition:**  
**Where it is used in the project:** The encrypted frame data becomes ciphertext before it is sent across the network.
The encrypted form of the original data.

---
## Symmetric Encryption
**Definition:**  
Encryption where the same secret key, or closely related secret material, is used for encryption and decryption.

**Where it is used in the project:** It is the appropriate production mechanism for encrypting the actual video payload after a session key is established.
**Examples:**  
AES and ChaCha20.

**Why it is good for video:**  
Symmetric encryption is much faster than public-key encryption for large amounts of data.

---
## Asymmetric Cryptography
**Definition:**  
Cryptography using a public key and a private key.

```text
Public Key  → Can be shared
Private Key → Must remain secret
```

**Where it is used in the project:** It is used for RSA and public/private-key operations during the security handshake.
**Examples:**  
RSA and public-key operations associated with modern key-establishment systems.

---
## Public Key
**Definition:**  
**Where it is used in the project:** Public keys are exchanged during the handshake so the parties can perform the required public-key operations.
A cryptographic key that can be shared with other parties.

---
## Private Key
**Definition:**  
**Where it is used in the project:** Private keys remain on the appropriate endpoint and are used for private-key cryptographic operations.
A secret cryptographic key that must be protected by its owner.

---
## Key Pair
**Definition:**  
**Where it is used in the project:** RSA public/private key pairs are part of the project's cryptographic setup.
The combination of a public key and its corresponding private key.

---
## Key Exchange / Key Establishment
**Definition:**  
The process of enabling two parties to obtain shared secret material securely.

**Where it is used in the project:**  
Kyber is used to establish a shared secret.

---

## Shared Secret
**Definition:**  
Secret key material known by both communicating parties but not by an attacker.

**Where it is used in the project:**  
The Kyber operation produces a 32-byte shared secret.

---

## KEM
**Definition:**  
Key Encapsulation Mechanism.

A KEM is designed to establish shared secret key material using public-key cryptography.

```text
Public Key
    ↓
Encapsulation
    ↓
Ciphertext + Shared Secret

Private Key + Ciphertext
    ↓
Decapsulation
    ↓
Same Shared Secret
```

**Important:**  
A KEM is not normally used to encrypt an entire video stream directly.

---

## AEAD
**Definition:**  
Authenticated Encryption with Associated Data.

It provides encryption plus integrity/authentication for encrypted data.

**Where it is used in the project:** It is the recommended production replacement for the prototype's insecure repeating-XOR payload encryption.
**Examples:**  
AES-GCM and ChaCha20-Poly1305.

**Why preferred for video payloads:**  
It is fast and protects both confidentiality and tampering detection.

---
## Nonce
**Definition:**  
A number used once with a cryptographic operation.

**Where it is used in the project:** It is required by the recommended production AEAD design to ensure safe per-message encryption.
**Why important:**  
Many modern authenticated-encryption schemes require a unique nonce for each encryption operation under the same key.

---
## IV
**Definition:**  
Initialization Vector. Additional input used by some encryption modes to make encryption of repeated data produce different ciphertext.

**Where it is used in the project:** It is relevant to the per-encryption initialization data required by the selected cipher/mode.
**Important:**  
The exact requirements depend on the cryptographic algorithm and mode.

---

# 4. Post-Quantum Cryptography Terms
## Post-Quantum Cryptography (PQC)
**Definition:**  
Cryptography designed to remain secure against attackers using sufficiently powerful quantum computers.

**Where it is used in the project:** The project demonstrates PQC through Kyber/ML-KEM key establishment.
**Simple explanation:**  
It prepares systems for a future where some current public-key algorithms may be broken by quantum algorithms.

---
## Quantum Computer
**Definition:**  
**Where it is used in the project:** It provides the future-threat motivation for including post-quantum cryptography in the design.
A type of computer based on quantum-mechanical principles that can solve certain classes of problems differently from classical computers.

---
## Shor's Algorithm
**Definition:**  
A quantum algorithm that can efficiently solve integer factorization and discrete logarithm problems.

**Where it is used in the project:** It motivates the project's concern about the long-term security of classical RSA-style public-key cryptography.
**Why important:**  
Those problems underpin the security of important classical public-key systems such as RSA and ECC.

---
## Harvest Now, Decrypt Later
**Definition:**  
An attack strategy where an adversary records encrypted data today and attempts to decrypt it in the future when better capabilities become available.

**Where it is used in the project:** It explains why encrypted streaming data with long-term sensitivity may benefit from PQC.
**Why PQC matters:**  
Long-lived sensitive data may need protection against future quantum attacks.

---
## Kyber / ML-KEM
**Definition:**  
Kyber is the name historically associated with a lattice-based post-quantum KEM. The standardized NIST version is called ML-KEM.

**In the project:**  
The code uses the Kyber512 parameter set through `liboqs`.

---

## Module-LWE
**Definition:**  
Module Learning With Errors is a mathematical hardness problem used as a foundation for lattice-based cryptography.

**Where it is used in the project:** It is part of the mathematical foundation behind the Kyber/ML-KEM algorithm used by the project.
**Simple explanation:**  
It is designed to create mathematical problems that are difficult to solve efficiently, including for known quantum attack methods.

---
## Lattice-Based Cryptography
**Definition:**  
A family of cryptographic techniques based on mathematical problems involving lattices.

**Where it is used in the project:** Kyber/ML-KEM belongs to this family and is the project's post-quantum approach.
**Why important:**  
It is one of the major approaches used in post-quantum cryptography.

---

# 5. RSA & OpenSSL Terms
## RSA
**Definition:**  
RSA is a classical asymmetric cryptographic algorithm based on the difficulty of factoring large integers.

**Where it is used in the project:**  
RSA-2048 is used for public-key operations and payload encryption in the prototype.

**Production note:**  
RSA should generally not be used for bulk video encryption.

---

## RSA-2048
**Definition:**  
RSA using a 2048-bit modulus.

**Important size:**  
2048 bits = 256 bytes.

Therefore, a normal RSA-2048 ciphertext block is 256 bytes.

---

## OAEP
**Definition:**  
Optimal Asymmetric Encryption Padding.

It is a padding scheme used with RSA encryption to improve security and randomness.

**Where it is used in the project:**  
RSA encryption uses OAEP.

---

## RSA Chunk Size
**Definition:**  
Because RSA can only encrypt a limited amount of data in one operation, large data must be divided into smaller chunks.

For RSA-2048 with the OAEP configuration described in the project:

```text
RSA block = 256 bytes
Padding overhead = 42 bytes
Maximum plaintext = 256 - 42 = 214 bytes
```

So the project uses 214-byte plaintext chunks.

---

## Digital Signature
**Definition:**  
A cryptographic mechanism used to prove that data was produced or approved by the holder of a private key and that the signed data has not been altered.

**Where it is used in the project:** It is relevant to stronger endpoint authentication, although the prototype primarily demonstrates encryption and key establishment.
**Important distinction:**  
Encryption protects confidentiality; signatures primarily provide authenticity and integrity.

---
## OpenSSL
**Definition:**  
A widely used cryptographic and TLS library.

**Where it is used in the project:**  
It provides RSA, TLS, certificates, and other cryptographic functionality.

---

## liboqs
**Definition:**  
An open-source library providing implementations of post-quantum cryptographic algorithms through a C API.

**Where it is used in the project:**  
It is used to access the Kyber KEM implementation.

---

## PEM
**Definition:**  
Privacy-Enhanced Mail is a text-based encoding commonly used to store cryptographic keys and certificates.

A PEM file typically contains readable headers such as:

```text
-----BEGIN PUBLIC KEY-----
...
-----END PUBLIC KEY-----
```

---

## X.509 Certificate
**Definition:**  
A standardized digital certificate format used to bind an identity to a public key.

**In HTTPS:**  
The server presents a certificate to the client.

---

## Self-Signed Certificate
**Definition:**  
A certificate signed by its own private key rather than by a trusted Certificate Authority.

**Use:**  
Common in development and testing.

**Production concern:**  
Clients need a proper trust mechanism to verify the server's identity.

---

## Certificate Verification
**Definition:**  
The process of checking whether a presented TLS certificate is trusted and valid.

**Where it is used in the project:** It is required when the client verifies the server's TLS identity to prevent server impersonation.
**Why important:**  
Without proper verification, encryption alone does not prevent a man-in-the-middle attacker from impersonating the server.

---

# 6. Video & Image Processing Terms
## OpenCV
**Definition:**  
An open-source computer vision library used for image and video processing.

**Where it is used in the project:**  
It handles camera capture, JPEG encoding/decoding, image manipulation, and related operations.

---

## Webcam Capture
**Definition:**  
**Where it is used in the project:** It is the first stage of the video pipeline and supplies frames to OpenCV for processing.
Reading video frames from a camera device.

```text
Camera
  ↓
VideoCapture
  ↓
cv::Mat
```

---
## cv::Mat
**Definition:**  
OpenCV's main data structure for storing images and matrices.

**Where it is used in the project:**  
Camera frames are initially represented as `cv::Mat`.

---

## BGR
**Definition:**  
**Where it is used in the project:** It represents the color format used by OpenCV image/frame data before display conversion when needed.
A color-channel ordering where the channels are Blue, Green, and Red.

OpenCV commonly represents color images in BGR order.

---
## BGRA
**Definition:**  
**Where it is used in the project:** It is relevant when preparing image data for Windows graphics display.
Blue, Green, Red, Alpha.

The additional Alpha channel is often used when working with graphics APIs.

---
## JPEG
**Definition:**  
A lossy image compression format commonly used for photographs and video-frame compression.

**Where it is used in the project:** Each camera frame is compressed into JPEG data before it is encrypted and transmitted.
**Why StreamOn uses it:**  
A raw camera frame is much larger than its JPEG representation.

---
## JPEG Compression
**Definition:**  
**Where it is used in the project:** It reduces the amount of data that the cryptography and network layers need to process.
Reducing the size of an image by removing information that is less important for visual perception.

```text
Raw Frame
   ↓
JPEG Compression
   ↓
Much Smaller Frame
   ↓
Network Transmission
```

---
## JPEG Quality
**Definition:**  
A parameter controlling the balance between image quality and compressed file size.

**Where it is used in the project:** It controls the frame-size versus visual-quality trade-off in the streaming pipeline.
**General rule:**

```text
Higher Quality → Larger Data → Better Image
Lower Quality  → Smaller Data → More Compression
```

---
## cv::imencode
**Definition:**  
An OpenCV function that encodes an image into a compressed format such as JPEG.

---

## cv::imdecode
**Definition:**  
An OpenCV function that converts encoded image bytes back into an image.

---

## FourCC
**Definition:**  
Four Character Code used to identify video codecs or formats.

**Where it is used in the project:**  
`MJPG` is used for Motion JPEG recording.

---

## MJPEG
**Definition:**  
Motion JPEG represents a video as a sequence of independently compressed JPEG images.

**Advantage:**  
Each frame is relatively independent.

**Disadvantage:**  
It can require more storage/bandwidth than inter-frame video codecs.

---

# 7. C++ & Concurrency Terms

## Thread
**Definition:**  
A separate path of execution within a program.

**Where it is used in the project:**  
Different threads handle UI, camera capture, and networking.

---

## Main/UI Thread
**Definition:**  
The thread responsible for processing the application's user-interface events and drawing UI elements.

**Where it is used in the project:** It handles Windows messages and rendering while worker threads perform blocking or expensive work.
**Important rule:**  
It should not be blocked by expensive network or cryptographic work.

---
## Mutex
**Definition:**  
**Where it is used in the project:** It protects shared frame/state data that can be accessed by multiple threads.
A synchronization mechanism that allows only one thread at a time to access protected shared data.

```text
Thread A ──┐
           ↓
         Mutex
           ↓
      Shared Data
           ↑
Thread B ──┘
```

---
## std::lock_guard
**Definition:**  
A C++ RAII utility that automatically locks a mutex when created and unlocks it when it goes out of scope.

**Where it is used in the project:** It provides safe scoped locking around shared data in multithreaded sections.
**Benefit:**  
It reduces the chance of forgetting to unlock a mutex.

---
## Race Condition
**Definition:**  
A bug where the result depends on the unpredictable timing of multiple threads accessing shared data.

**Where it is used in the project:** It is a concurrency problem the project's mutexes and atomic state variables are intended to prevent.
**Example:**  
One thread writes a frame while another thread reads the same frame at the same time.

---
## Atomic
**Definition:**  
A variable that can be safely accessed by multiple threads for supported atomic operations without requiring a traditional mutex for each operation.

**Where it is used in the project:**  
Atomic Boolean flags are used for state such as running/connected/streaming.

---

## std::atomic<bool>
**Definition:**  
**Where it is used in the project:** It is used for flags such as running, connected, and streaming state.
A thread-safe Boolean variable suitable for simple shared state signaling between threads.

---
## Callback
**Definition:**  
A function supplied to another component so that it can be called later when an event occurs.

**Where it is used in the project:**  
A camera component can call a callback when a new frame is ready.

---

## Producer-Consumer Pattern
**Definition:**  
**Where it is used in the project:** The camera produces frames while processing, networking, or display components consume them.
A concurrency pattern where one component produces data and another consumes it.

```text
Camera
 Producer
    ↓
 Frame Buffer
    ↓
Consumer
 Network / UI
```

---
## PostMessage
**Definition:**  
A Windows API function that places a message into a window's message queue without waiting for the receiver to process it.

**Why useful:**  
A worker thread can notify the UI thread without directly performing UI drawing.

---

## RAII
**Definition:**  
Resource Acquisition Is Initialization.

C++ objects acquire resources during construction and release them automatically during destruction.

**Where it is used in the project:** It is used through C++ resource-management patterns such as automatic mutex unlocking.
**Examples:**  
- Mutex locking with `std::lock_guard`
- Resource cleanup in destructors

---
## Memory Leak
**Definition:**  
Memory that is allocated but never properly released.

**Why dangerous:**  
Repeated leaks can cause increasing memory usage and eventually crash or degrade the application.

---

## Buffer
**Definition:**  
**Where it is used in the project:** Buffers hold camera frames, JPEG bytes, encrypted data, and network payloads while they move through the pipeline.
A temporary memory area used to hold data while it is being processed or transferred.

---
## std::vector<uint8_t>
**Definition:**  
A dynamically sized C++ array commonly used to store raw bytes.

**Where it is used in the project:**  
It stores JPEG data, ciphertext, keys, and network payloads.

---

# 8. Windows & UI Terms

## Win32 API
**Definition:**  
**Where it is used in the project:** It provides the native Windows application, window, message, and graphics functionality.
The native Windows programming interface used to create windows, handle events, access system functionality, and interact with graphics APIs.

---
## HWND
**Definition:**  
**Where it is used in the project:** It identifies the native Windows window/control used by the application's video UI.
A Windows handle that identifies a window or UI control.

---
## GDI
**Definition:**  
Graphics Device Interface, a Windows API used for drawing graphics and text.

**Where it is used in the project:**  
It is used to display video frames inside a native Win32 control.

---

## HDC
**Definition:**  
**Where it is used in the project:** It provides the drawing context used by GDI to render the video.
Handle to a Device Context.

A device context represents a drawing surface or graphics environment used by Windows GDI.

---
## SetDIBitsToDevice
**Definition:**  
A Windows GDI function that copies bitmap data directly to a device context.

**Where it is used in the project:**  
It is used to render decoded video frames inside the Win32 UI.

---

## DIB
**Definition:**  
**Where it is used in the project:** It is the bitmap representation used when preparing decoded frames for Windows GDI display.
Device Independent Bitmap, a bitmap format used by Windows for representing image data independently of a particular display device.

---
## Event Loop
**Definition:**  
**Where it is used in the project:** The Windows message loop processes UI events while the application is running.
A loop that continuously waits for and processes application events.

```text
Wait for Event
      ↓
Receive Event
      ↓
Process Event
      ↓
Update UI
      ↓
Wait Again
```

---

# 9. Security Terms & Vulnerabilities
## Confidentiality
**Definition:**  
Ensuring that unauthorized people cannot read the data.

**Where it is used in the project:** It is provided by encrypting the video payload before network transmission.
**Example:**  
Encrypting a video frame.

---
## Integrity
**Definition:**  
Ensuring that data has not been modified without detection.

**Where it is used in the project:** It is a production security requirement that should be provided by authenticated encryption.
**Example:**  
Authenticated encryption can detect tampering.

---
## Authentication
**Definition:**  
Verifying the identity of the communicating party.

**Where it is used in the project:** It is provided by proper TLS certificate verification and stronger endpoint-authentication mechanisms.
**Example:**  
TLS certificate verification helps authenticate a server.

---
## Man-in-the-Middle (MITM) Attack
**Definition:**  
An attacker secretly positions themselves between two communicating parties and attempts to intercept or modify their communication.

```text
Client ←→ Attacker ←→ Server
```

**Where it is used in the project:** It is a threat addressed by authenticated TLS rather than relying on encryption alone.
**Protection:**  
Proper TLS certificate verification and authenticated cryptographic protocols.

---
## Known-Plaintext Attack
**Definition:**  
An attack where an attacker knows or can predict some plaintext and uses the corresponding ciphertext to learn information about the encryption.

**Where it is used in the project:** It explains why predictable JPEG bytes make the prototype's repeating-XOR scheme insecure.
**Why relevant to StreamOn:**  
JPEG files have predictable headers, which makes a simple repeating XOR design unsafe.

---
## XOR
**Definition:**  
**Where it is used in the project:** It appears in the prototype's simple encryption experiment but is not suitable as production encryption.
Exclusive OR, a bitwise operation commonly represented by `^`.

Basic property:

```text
A XOR B XOR B = A
```

**Security warning:**  
XOR itself is not a secure encryption algorithm. Security depends on how it is used.

---
## Repeating-Key XOR
**Definition:**  
XORing data with a short key repeatedly.

```text
Data:  ABCDEFGHIJKLMNOP...
Key:   KEYKEYKEYKEYKEY...
```

**Where it is used in the project:** It is the prototype's insecure payload-encryption approach that should be replaced in production.
**Problem:**  
Repeating the key creates patterns and can make cryptanalysis possible.

---
## Key Reuse
**Definition:**  
Using the same cryptographic key repeatedly when the algorithm requires fresh key material or unique nonces.

**Where it is used in the project:** It is a concern when designing session keys and nonces for repeated video-frame encryption.
**Why dangerous:**  
It can expose relationships between different encrypted messages.

---
## Vulnerability
**Definition:**  
**Where it is used in the project:** The main example in the project is the insecure repeating-XOR payload encryption.
A weakness in a system that can potentially be exploited by an attacker.

---
## Security Trade-off
**Definition:**  
A design choice where improving one property may affect another property such as performance, simplicity, or security.

**Example:**  
UDP can reduce latency but does not provide reliable delivery by itself.

---

# 10. Performance & Optimization Terms

## Latency
**Definition:**  
**Where it is used in the project:** It is a key streaming metric because the application aims to keep displayed video close to the live camera feed.
The delay between an action occurring and its result being observed.

**For video:**  
Camera captures frame → client displays frame.

The elapsed time is part of streaming latency.

---
## Throughput
**Definition:**  
The amount of data that can be processed or transferred per unit of time.

**Where it is used in the project:** It determines how quickly frames can be compressed, encrypted, transmitted, decrypted, and displayed.
**Example:**  
Megabits per second (Mbps).

---
## FPS
**Definition:**  
Frames Per Second.

**Where it is used in the project:** It describes the rate at which camera frames are captured and streamed.
**Example:**  
30 FPS means approximately 30 video frames are processed each second.

---
## CPU Bottleneck
**Definition:**  
A situation where CPU processing becomes the limiting factor preventing the application from achieving better performance.

**Where it is used in the project:** It is especially relevant to the prototype because RSA operations on video chunks can be computationally expensive.
**In the project:**  
Performing RSA operations on every video-frame chunk can create a major CPU cost.

---
## Microbenchmark
**Definition:**  
A small performance measurement focused on a particular operation.

**Where it is used in the project:** It can be used to measure the cost of frame processing and cryptographic operations in the project.
**Example:**  
Measuring the time required to encrypt one frame.

---
## std::chrono
**Definition:**  
**Where it is used in the project:** It can be used to measure frame-processing and cryptographic timings.
A C++ standard library facility for working with clocks, time points, durations, and timing measurements.

---
## Microsecond
**Definition:**  
One millionth of a second.

```text
1 second = 1,000,000 microseconds
```

---

## Zero-Copy
**Definition:**  
A design approach that minimizes or eliminates unnecessary copying of data between memory buffers.

**Where it is used in the project:** It is a performance optimization goal for reducing unnecessary frame/data copies between pipeline stages.
**Why useful:**  
Less copying can reduce CPU usage and memory bandwidth consumption.

---
## Backpressure
**Definition:**  
**Where it is used in the project:** It becomes relevant when frames are produced faster than the network or client can consume them.
A situation where the producer generates data faster than the consumer can process or transmit it.

```text
Camera: 60 FPS
   ↓
Network: 10 FPS
   ↓
Backpressure
```

---
## Frame Dropping
**Definition:**  
Deliberately discarding some frames when the system cannot process or transmit all of them.

**Where it is used in the project:** It is a real-time strategy for preventing old frames from building up when the consumer falls behind.
**Why useful for live video:**  
Showing the newest frame is often better than displaying very old frames.

---
## Latest-Frame Strategy
**Definition:**  
Keeping only the newest frame instead of maintaining a large queue of old frames.

**Where it is used in the project:** It keeps the newest frame available instead of maintaining an ever-growing queue of stale frames.
**Benefit:**  
Memory remains bounded and the displayed video stays temporally fresh.

---

# 11. Protocol & Data Format Terms
## JSON
**Definition:**  
JavaScript Object Notation, a lightweight text-based format for structured data.

**Example:**

```json
{
  "rsa_public_key": "...",
  "kyber_public_key": "..."
}
```

**Where it is used in the project:**  
Used for handshake information and public-key exchange.

---

## Endpoint
**Definition:**  
**Where it is used in the project:** It identifies the HTTP operations exposed by the server, such as key, handshake, and frame operations.
A specific network URL/path through which a service exposes an operation.

**StreamOn examples:**

```text
GET  /keys
POST /handshake
GET  /frames
```

---
## GET
**Definition:**  
An HTTP method generally used to request information from a server.

**Where it is used in the project:** It is used for retrieving information such as keys or frames.
**Example:**  
`GET /frames`

---
## POST
**Definition:**  
An HTTP method generally used to send data to a server for processing.

**Where it is used in the project:** It is used for sending handshake or related data to the server.
**Example:**  
`POST /handshake`

---
## HTTP Status Code
**Definition:**  
A numeric code returned by an HTTP server describing the result of a request.

**Where it is used in the project:** It communicates the result of HTTP requests in the project's client-server communication.
**Example:**  
`204 No Content` means the request succeeded but there is currently no content to return.

---
## Content-Type
**Definition:**  
An HTTP header describing the format of the data being sent.

**Where it is used in the project:** It identifies whether an HTTP response contains JSON, JPEG, or binary data.
**Example:**

```text
application/octet-stream
```

means arbitrary binary data.

---
## Binary Data
**Definition:**  
Raw bytes rather than human-readable text.

**Where it is used in the project:**  
Encrypted video frames are binary data.

---

## Serialization
**Definition:**  
Converting structured information into a format suitable for storage or transmission.

**Where it is used in the project:** It converts structured handshake or packet information into a transferable representation.
**Example:**  
Converting packet fields into a sequence of bytes.

---
## Deserialization
**Definition:**  
**Where it is used in the project:** It reconstructs structured information from received JSON or binary packet data.
Converting serialized bytes back into structured information.

---
## Header
**Definition:**  
**Where it is used in the project:** Custom UDP headers carry metadata needed to interpret and reassemble encrypted frame chunks.
Metadata placed before the main payload to describe how the payload should be interpreted.

**StreamOn UDP packet:**

```text
Header + Encrypted Payload
```

---
## Endianness
**Definition:**  
The order in which the bytes of a multi-byte number are stored or transmitted.

**Big-endian:**  
Most significant byte first.

**Where it is used in the project:**  
Manual bit shifts are used to construct multi-byte packet fields in a consistent byte order.

---

# 12. Interview Quick-Revision Sheet

## If the interviewer asks: "What is StreamOn?"
> "StreamOn is a C++ secure webcam streaming prototype. It captures video, compresses it, encrypts it, sends it over the network, and decrypts and displays it on the client."

---

## If asked: "Why Kyber?"
> "Kyber is a post-quantum key encapsulation mechanism. It is used to establish shared secret material that is designed to resist known quantum attacks."

---

## If asked: "Why not encrypt the video with Kyber?"
> "Kyber is a KEM, not a bulk-data encryption algorithm. It is used to establish a shared secret, while a symmetric cipher such as AES-GCM should encrypt the actual video."

---

## If asked: "Why is RSA bad for video encryption?"
> "RSA is computationally expensive and can only encrypt small chunks at a time. For every video frame, many RSA operations would be required. A symmetric cipher is much faster for bulk video data."

---

## If asked: "Why JPEG before encryption?"
> "Raw video frames are very large. JPEG compression significantly reduces the amount of data that needs to be encrypted and transmitted."

---

## If asked: "TCP vs UDP?"
> "TCP provides reliable and ordered delivery, but retransmissions can increase latency. UDP has lower overhead and is better suited to real-time media where dropping an occasional frame can be preferable to waiting for an old packet."

---

## If asked: "What is the biggest security issue in the UDP prototype?"
> "The repeating XOR encryption is not secure. JPEG has predictable bytes, so an attacker can use known plaintext to recover key information. It should be replaced with authenticated encryption such as AES-GCM or ChaCha20-Poly1305."

---

## If asked: "Why certificate verification?"
> "TLS encrypts the connection, but the client also needs to verify that it is actually talking to the intended server. Otherwise, a man-in-the-middle attacker could impersonate the server."

---

## If asked: "How do you prevent race conditions?"
> "Shared frame data is protected with mutexes, simple state flags use atomics, and the camera thread sends UI notifications instead of directly drawing on the UI thread."

---

## If asked: "How do you handle slow networks?"
> "The prototype keeps the latest frame and overwrites older frames. This keeps memory bounded and prioritizes live video freshness over displaying every frame."

---

## If asked: "What is the production-ready encryption design?"
> "Use RSA or post-quantum key establishment only during the handshake, derive a session key, and then encrypt the video stream with a fast authenticated symmetric cipher such as AES-256-GCM or ChaCha20-Poly1305."

---

# Final Mental Model

The easiest way to remember the entire project is:

```text
                  STREAMON
                     │
        ┌────────────┴────────────┐
        ↓                         ↓
     Camera                    Security
        │                         │
        ↓                         ↓
     OpenCV                 Kyber + RSA/TLS
        │                         │
        ↓                         ↓
   JPEG Compression        Key Establishment
        │                         │
        └────────────┬────────────┘
                     ↓
                 Encryption
                     ↓
                  Network
              ┌──────┴──────┐
              ↓             ↓
             TCP           UDP
              │             │
              └──────┬──────┘
                     ↓
                 Decryption
                     ↓
                JPEG Decode
                     ↓
                 Win32 UI
                     ↓
                 Video Output
```

**One-line project summary:**

> **Capture → Compress → Establish Keys → Encrypt → Transport → Decrypt → Decode → Display**
