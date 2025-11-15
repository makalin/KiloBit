# **KiloBit**

**KiloBit: A high-performance, schema-driven binary serialization format designed for extreme compactness.**

KiloBit is a conceptual project to create a new binary data format that prioritizes minimal byte size and high-speed, low-overhead parsing.

## **The Problem**

Standard data formats all have trade-offs:

* **JSON:** Human-readable and ubiquitous, but extremely verbose and slow to parse. File sizes are large due to redundant keys, whitespace, and text-based numbers.  
* **MessagePack/CBOR:** These "schema-less" binary formats are a huge improvement on JSON. However, they still repeat string keys for every object, wasting space in large datasets.  
* **Protocol Buffers/Cap'n Proto:** These "schema-based" formats are extremely fast and compact. Their main drawback is operational overhead: they require managing separate .proto or .capnp schema files and integrating a "build step" into your workflow to generate code.  
* **TOON:** This modern, text-based format is brilliant for reducing *LLM token counts*, but it is not a binary format and is not optimized for raw *byte size*.

## **The KiloBit Solution**

KiloBit aims to be the best of all worlds by using a **self-contained, embedded schema**.

It's a binary format that operates like Protocol Buffers (using integer IDs for keys) but without the need for external schema files. The "schema" is just a simple dictionary, or "key table," embedded *once* at the beginning of the file.

### **How it Works: File Architecture**

A KiloBit file is composed of three main parts:

1. **File Header (2 bytes):**  
   * A "magic number" (0xKB) to identify the file as KiloBit.  
   * A version byte.  
2. **Dictionary Header (Schema):**  
   * A simple table mapping all string keys used in the document to short, variable-length integers (varints).  
   * **Example:** \[1: "userId", 2: "firstName", 3: "email", 4: "orders"\]  
   * This section is written only once at the start of the file.  
3. **Data Body:**  
   * The rest of the file contains the data, serialized in a compact binary form.  
   * Instead of writing the string "userId", KiloBit just writes its dictionary ID, 1\.  
   * This eliminates 100% of key string repetition, leading to massive size savings on large arrays of objects.

#### **Example Comparison**

Imagine this JSON data (110 bytes):

\[  
  { "userId": 1, "email": "a@dv.com.tr" },  
  { "userId": 2, "email": "b@dv.com.tr" }  
\]

A **KiloBit** representation would look like this conceptually:

\[FILE HEADER: 0xKB01\]  
\[DICTIONARY: (1:"userId", 2:"email")\]  
\[DATA BODY:  
  (ARRAY\_MARKER, 2\_ITEMS)  
    (OBJECT\_MARKER)  
      (KEY\_ID:1, INT:1)  
      (KEY\_ID:2, STR:"a@dv.com.tr")  
    (OBJECT\_MARKER)  
      (KEY\_ID:1, INT:2)  
      (KEY\_ID:2, STR:"b@dv.com.tr")  
\]

The strings "userId" and "email" are stored *only once*, no matter if there are two objects or two million.

## **Why KiloBit?**

* **Extreme Compactness:** Aims to be significantly smaller than Gzip'd JSON and more compact than MessagePack for large, repetitive datasets.  
* **High-Speed Performance:** Parsing is extremely fast. Reading a key is an integer lookup, not a string hash or comparison.  
* **Self-Contained:** A KiloBit file is fully portable. You don't need a separate .schema file to read it, making it perfect for APIs, file storage, and network transport.  
* **Flexibility:** Ideal for everything from game state snapshots and IoT data streams to API responses and document storage.

## **Project Status: Conceptual**

**KiloBit is currently in the conceptual design phase.** The specification is not yet final, and there are no official reference implementations.

This is the perfect time to get involved and help shape the future of a high-performance data format.

## **How to Contribute**

We are actively looking for collaborators to help turn this idea into a reality.

* **Help Define the Spec:** Join the discussion (in Issues) to refine the data types, header structure, and encoding rules.  
* **Build a Reference Implementation:** We need libraries\! The first step is a reference implementation, likely in a systems language like **Rust** or **Go**.  
* **Build Client Libraries:** Once the spec is stable, creating libraries for **Python**, **TypeScript/JavaScript**, **C\#**, and **Java** will be critical for adoption.

## **License**

This project, including the specification and any future reference code, is licensed under the **MIT License**.
