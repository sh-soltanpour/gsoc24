<img src="https://developers.google.com/open-source/gsoc/resources/downloads/GSoC-Vertical.svg" alt="GSoC logo" width="80" /> <img src="https://www.postgresql.org/media/img/about/press/elephant.png" alt="PostgreSQL logo" width="80" />
## Google Summer of Code 2024 Final Report

#### **Project Title**: pgmoneta - WAL Infrastructure  
#### **Contributor**: Shahryar Soltanpour  
#### **Organization**: [PostgreSQL](https://www.postgresql.org/)  
#### **Mentors**: Jesper Pedersen, Haoran Zhang  
#### **Progress Report**: https://github.com/pgmoneta/pgmoneta/discussions/288
#### **Contact**: [shahryar.soltanpour@gmail.com](mailto:shahryar.soltanpour@gmail.com)  
#### **LinkedIn**: [Shahryar Soltanpour](https://www.linkedin.com/in/soltanpour/)

---

## Overview

My project, **pgmoneta: WAL Infrastructure**, focused on adding a powerful Write-Ahead Log (WAL) system to pgmoneta. This setup provides detailed insights into WAL files, so in future it can be used as the backbone of precise point-in-time recovery (PiTR). In the future, we might even expand to analyze WAL data at a logical level, giving even deeper insights into changes.

The results of my project will be available starting with release 0.15.x of pgmoneta and forward.

For more info on the project’s beginnings, check out [my initial proposal](https://docs.google.com/document/d/194bAxMbiOvQhL4rCH30mlIhHZW9UccbQa2F8EEMCIwI/).

---

## Key Achievements

### 1. WAL Parsing Framework

I built a WAL parsing system compatible with PostgreSQL versions 13 through 17, making sure it handles differences across versions for wide compatibility. 

**Why It’s Important**: This allows pgmoneta to read and interpret transaction logs, making it adaptable across PostgreSQL versions.
**How It Works**: Built using a factory pattern and structured in a way that makes it easy to adapt to different WAL structures. I compiled [a detailed list of WAL changes](https://elastic-traffic-761.notion.site/Postgres-WAL-diff-5d7f873c9d204ff79cb6779262c76326?pvs=74) to guide development.

**Pull Request**: Parsing system developed in [PR #355](https://github.com/pgmoneta/pgmoneta/pull/355).

### 2. WAL Write Functionality

Developed the ability to reassemble parsed WAL records into new WAL files, tested by comparing with the originals to ensure accuracy.

**Why It’s Important**: By re-creating the WAL files, we can read a file from a Postgres instance and write it for another instance or version. 
**How It Works**: Currently there's a high-level function in pgmoneta that gets the source file and generates the output file, by parsing it and rewriting it.

**Pull Request**: Writing functionality developed in [PR #381](https://github.com/pgmoneta/pgmoneta/pull/381).

### 3. Command-Line Tool - `pgmoneta-walinfo`

I created a CLI tool that gives PostgreSQL admins detailed views into WAL files. Designed with flexible output options and decompression, making it easy to use for detailed file inspection. Key features include:

- **Decompression**  
- **Output options in JSON and raw text**  
- **Color-coded displays**  
- **Handling partial records**  


**Why It’s Important**: The tool provides JSON and RAW output for script integration and color-coding to help with quick, manual or automated inspections.
**How It Works**: a usage sample is: `pgmoneta-walinfo <file> --format json`


**Pull Request**: CLI tool implemented in [PR #399](https://github.com/pgmoneta/pgmoneta/pull/399).

---

## Key Challenges and How I Overcame Them

### 1. **Ensuring Compatibility Across PostgreSQL Versions**

One of the most significant challenges was ensuring that our infrastructure could handle the various WAL formats from PostgreSQL versions 13 through 17. Each version had slight but critical differences in structure, fields, and identifiers, as mentioned above, I compiled [a detailed list of WAL changes](https://elastic-traffic-761.notion.site/Postgres-WAL-diff-5d7f873c9d204ff79cb6779262c76326?pvs=74) to guide development for official "magic values" versions (e.g., 0xD106 for PostgreSQL 13 and 0xD116 for PostgreSQL 17). Changes in data structuring, such as the handling of certain flags, affected how records needed to be parsed and written.

To address this, I created a factory pattern within the parser that dynamically adjusted to each version's unique requirements. Using version-specific flags and identifiers, the parser could identify the correct struct formats and apply version-based parsing methods. This adaptive structure allowed us to seamlessly support multiple versions, saving us from duplicating code while maintaining flexibility for future PostgreSQL updates.

### 2. **Managing Complex Data Alignments and Record Boundaries**

Parsing WAL records that spanned multiple pages was technically challenging. Each WAL record has a fixed header, but the data portion could span across page boundaries, which required precise handling to avoid misalignment. Misalignments could lead to incorrect data parsing, causing errors in the encoding or even failing to retrieve subsequent records. Initially, my attempts to align records using basic offset adjustments were unsuccessful, as the boundary offsets could vary based on record type and PostgreSQL version.

Implementing this, along with buffer management techniques, allowed me to accurately parse records across boundaries without losing any part of the data. Additionally, I developed error-handling mechanisms to detect and correct any anomalies in boundary alignment dynamically, ensuring seamless parsing through the entire WAL file.

### 3. **Handling Memory Allocation Across Platforms**

Memory leaks were a persistent challenge. Using Valgrind, I resolved allocation issues to ensure stable performance across platforms.

### 4. **Building a Clear, Modular API for WAL Parsing and Writing**

Given the complexity of WAL operations, creating a clear and modular API was crucial. The API needed to accommodate varying PostgreSQL versions, support both reading and writing operations and integrate seamlessly with pgmoneta’s existing infrastructure. However, balancing simplicity with functionality was a challenge, especially as the project scope expanded to include writing functionality and CLI options for multiple output formats (raw and JSON).

To ensure clarity and modularity, I introduced `walfile.h` as a central header file that consolidated all WAL-related operations, including reading, writing, and format conversions. This file exposed only high-level functions necessary for external use, hiding the internal complexities from other parts of the codebase. I also added extensive inline documentation, which explained each function’s purpose, inputs, and outputs, making the API easier for others to use and modify. Through feedback from my mentors, I refined the API further, ensuring that it was both user-friendly and adaptable to future enhancements, such as adding new serialization formats or debug features.

### 5. **Implementing User-Friendly CLI**

The `pgmoneta-walinfo` CLI tool required a user-friendly design for developers and database administrators to inspect WAL files easily. The CLI needed to support various formats (raw and JSON), provide decompression for compressed WAL files, and offer color-coded outputs for enhanced readability. However, the challenge was to ensure that these features were implemented efficiently without overwhelming memory resources.

To solve this, I modularized the CLI’s output logic, separating format conversions into discrete functions. This structure allowed the CLI to convert records to JSON or raw format without loading the entire WAL file into memory. For color-coded raw output, I assigned each column a distinct color, making it easier to visually parse and interpret the output. Moreover, the JSON output was designed to print records incrementally, which avoided memory bloat. These adjustments made the CLI a lightweight yet powerful tool for WAL inspection, offering flexible output options to users.


## Future Improvements

Some ideas for expanding this project include:

1. **More Options for `pgmoneta-walinfo`**:  
   - Adding flags like `-o, --output`, `-q, --quiet`, `-L, --logfile`, and `--color on/off` would make it even more user-friendly.
   - Integrating filters similar to `pg_waldump` could allow for targeted data inspection.

2. **Cross-Version WAL Handling**:  
   - Enabling conversion of WAL files across PostgreSQL versions would make upgrades and migrations much easier.

3. **Extracting Logical WAL Data**:  
   - Expanding parsing capabilities to handle logical data for even deeper insights.

4. **Performance Optimization**:  
   - Speeding up parsing with worker processes and better memory usage for faster handling of large logs.

---

## Key Takeaways

This project has been a deep dive into database internals and C programming. Here are a few things I learned:

- **PostgreSQL WAL Compatibility**: Adapting to WAL changes across versions gave me a solid understanding of PostgreSQL’s logging and recovery mechanisms.
- **Systems Programming in C**: Writing efficient, adaptable code in C, especially for data handling, pushed me to think carefully about performance and memory.
- **Collaboration and Mentorship**: Working with Jesper and Haoran was a great learning experience, sharpening my skills in iterative development and problem-solving with a team.

---

## Thank You!

Big thanks to my mentors Jesper Pedersen and Haoran Zhang, and Google for this incredible Google Summer of Code opportunity. This project has been invaluable for my growth in database systems and open-source collaboration. I'm excited to keep contributing to pgmoneta and PostgreSQL!
