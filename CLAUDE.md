# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the libp2p specifications repository, containing protocol specifications, RFCs, and documentation for the libp2p peer-to-peer networking framework. This is primarily a documentation repository with markdown files rather than executable code.

## Architecture & Structure

### Core Framework Documents
- **Spec Lifecycle**: `00-framework-01-spec-lifecycle.md` - Defines the three-level maturity process (Working Draft → Candidate Recommendation → Recommendation) with status classifications (Active/Deprecated/Terminated)
- **Document Headers**: `00-framework-02-document-header.md` - Standardized header format for all libp2p specs including lifecycle stage, authors, and interest groups

### Major Protocol Categories
- **Core Abstractions**: `addressing/`, `connections/`, `peer-ids/` - Fundamental libp2p concepts
- **Transport Protocols**: `quic/`, `tcp/`, `webrtc/`, `webtransport/`, `websockets/` - Connection layer protocols  
- **Security Protocols**: `noise/`, `tls/`, `secio/`, `plaintext/` - Encryption and authentication
- **Stream Multiplexers**: `mplex/`, `yamux/` - Managing multiple streams over single connections
- **Discovery & Routing**: `kad-dht/`, `mdns/`, `rendezvous/` - Peer and content discovery mechanisms
- **NAT Traversal**: `autonat/`, `relay/`, `connections/hole-punching.md` - Connectivity solutions
- **Messaging**: `pubsub/` (including `gossipsub/`) - Publish/subscribe messaging protocols
- **Utilities**: `ping/`, `identify/`, `fetch/` - Network utility protocols

### Document Standards
Each specification follows the standardized header format with:
- Lifecycle stage (e.g., "1A", "2A", "3A") indicating maturity level and status
- Authors and Interest Group members with GitHub handles
- Reference to the lifecycle specification for context

### RFCs Directory
Contains formal Request for Comments documents:
- RFC 0001: Text representation of PeerIDs and CIDs
- RFC 0002: Signed Envelopes for authenticated data
- RFC 0003: Routing Records for verifiable address distribution

## Development Workflow

### No Build System
This repository contains only documentation - there are no build commands, linting tools, or test suites in the traditional sense. Changes are made directly to markdown files.

### Specification Process
- New specs start as Working Drafts requiring 3+ libp2p contributors as Interest Group
- Promotion requires implementation evidence and community review
- Final Recommendations need 2+ interoperable implementations

### File Organization
- Protocol specs are organized in topical directories (e.g., `pubsub/gossipsub/`)
- Supporting materials like diagrams go in protocol-specific directories
- Archive materials are kept in `_archive/` directory

## Key Concepts

### Specification Maturity Levels
1. **Working Draft (Level 1)**: Under development, self-directed by author
2. **Candidate Recommendation (Level 2)**: Technically complete with 1+ implementation  
3. **Recommendation (Level 3)**: 2+ interoperable implementations, supreme stage

### Protocol Design Principles
- Use protocol buffers (protobuf) for wire protocols (prefer proto3 for new specs)
- Security-first design with defense against various attack vectors
- Support for diverse network environments (mobile, IoT, constrained networks)
- Interoperability across multiple language implementations

### Documentation Standards
- Clear motivation and scope sections
- Technical specifications sufficient for interoperable implementations
- References to related protocols and dependencies
- Implementation status tracking where applicable

## Related Repositories
- Individual implementation roadmaps exist in language-specific repos (go-libp2p, js-libp2p, rust-libp2p)
- Test plans and compatibility testing in separate repositories
- Live documentation at https://docs.libp2p.io for user-facing content