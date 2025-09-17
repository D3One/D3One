# PDF Signature Verifier Module (C++ / Windows CryptoAPI)

This project is a replica of a module I developed as an entry for the **2013 Infotecs Academy Young Scientists Contest** in Russia. The goal was to create a DLL library for verifying digital signatures in PDF documents using the Windows CryptoAPI.

## Project Overview

In 2013, ensuring the integrity and authenticity of signed PDF documents was a critical task for businesses adopting electronic document management systems. This module provides a low-level C++ implementation for verifying cryptographic signatures embedded in PDFs, serving as a reference for integration into larger security suites.

## Key Features

*   **PDF Parsing:** Extracts signature data and signed content from PDF files (simulated in this demo).
*   **Cryptographic Verification:** Uses Windows CryptoAPI to verify signatures (RSA, DSA) and compute hashes (SHA-1, SHA-256).
*   **C++ Native Interface:** Designed for easy integration into C++ and .NET applications.
*   **Detailed Logging:** Provides clear status messages for each verification step.

## Technical Details

*   **Language:** C++
*   **Platform:** Windows (API compatible with Windows 7 and newer)
*   **Core APIs:** Windows CryptoAPI, Win32 API
*   **Architecture:** DLL library with a clean C++ interface

## Disclaimer

This is a **demonstration and educational project** created in 2024 to replicate the work done in 2013. It simulates the core functionality but does not include a complete PDF parser or support for all signature formats. The code serves as a historical artifact and a learning resource for understanding digital signature verification and Windows cryptography.

## Building the Project

1.  Open a Visual Studio Command Prompt (or use MSVC compiler).
2.  Navigate to the project directory.
3.  Run the build script: `build.bat`

## Usage

The `main.cpp` file contains a simple example of how to use the `PdfVerifier` class.

```cpp
#include "PdfVerifier.h"

int main() {
    PdfVerifier verifier;
    bool isVerified = verifier.VerifySignature(L"document.pdf");
    // Check result using verifier.GetVerificationResult()
    return 0;
}
```

## License

This project is licensed under the MIT License - see the LICENSE file for details.
```

