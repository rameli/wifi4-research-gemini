# Phase 2 Candidate Parameters

This report consolidates the findings from Phase 1 research (Web, PDF, and Training Knowledge) regarding 802.11n parameters relevant for long-distance links.

### aSlotTime
- **Source agents**: P-Deep-1, W1, W2, W3, K1
- **Layer**: PHY / MAC
- **Criticality**: Mandatory
- **Default value**: 9 µs (Short Slot), 20 µs (Long Slot) [P-Deep-1, W1]
- **Long-distance value**: 9 µs + aAirPropagationTime [P-Deep-1]; 9 µs + (Distance / c) [W1, W2, K1]
- **Description**: The time unit used by the MAC for backoff and channel access.
- **Usage context**: Used in DCF and EDCA for backoff counters; a STA decrements its backoff counter once per slot time when the medium is idle.
- **IEEE section/page**: 10.3.2.16 (p. 1669), 10.3.7 (p. 1680), 17.4.4 (p. 2846) [P-Deep-1]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None on function; minor differences in "Long-distance value" formula representation.

### aSIFSTime
- **Source agents**: P-Deep-1, W1, W3, K1
- **Layer**: PHY / MAC
- **Criticality**: Mandatory
- **Default value**: 16 µs (5 GHz / HT), 10 µs (2.4 GHz) [P-Deep-1, W1]
- **Long-distance value**: Fixed per PHY (10/16 µs) [P-Deep-1, W1, K1]
- **Description**: Short Interframe Space; the shortest interval between frame transmissions.
- **Usage context**: Used between a data frame and its acknowledgment (ACK), or between fragments of a frame.
- **IEEE section/page**: 10.3.2.3.3 (p. 1643), 17.4.4 (p. 2846) [P-Deep-1]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### aDIFSTime
- **Source agents**: P-Deep-1, W1, K1
- **Layer**: MAC
- **Criticality**: Mandatory
- **Default value**: 34 µs (5 GHz / HT Short Slot), 28 µs (2.4 GHz Short Slot) [P-Deep-1]
- **Long-distance value**: aSIFSTime + 2 * aSlotTime [P-Deep-1, W1, K1]
- **Description**: DCF Interframe Space; the minimum time the medium must be idle before a STA can transmit under DCF.
- **Usage context**: Used for regular data and management frame transmissions.
- **IEEE section/page**: 10.3.7 (p. 1681) [P-Deep-1]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### aEIFSTime
- **Source agents**: P-Deep-1, W1, K1
- **Layer**: MAC
- **Criticality**: Mandatory
- **Default value**: ~94 µs (typical for 20 MHz OFDM) [P-Deep-1, W1]
- **Long-distance value**: aSIFSTime + AckTxTime + DIFS [P-Deep-1, W1, K1]
- **Description**: Extended Interframe Space; used after the reception of a corrupted frame.
- **Usage context**: Prevents a STA from interfering with an ACK that might be sent by another STA that received the frame correctly.
- **IEEE section/page**: 10.3.7 (p. 1681) [P-Deep-1]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### dot11GuardInterval
- **Source agents**: P-Deep-1, W1, W3, W5, K1, K3
- **Layer**: PHY
- **Criticality**: Mandatory
- **Default value**: 800 ns (Long GI) [P-Deep-1, W1]
- **Long-distance value**: 800 ns (Long GI preferred for stability) [W1, W3, K1]
- **Description**: The circular prefix prepended to an OFDM symbol to mitigate Inter-Symbol Interference (ISI).
- **Usage context**: Used in all OFDM/HT transmissions by default; Short GI (400 ns) is an optional HT feature.
- **IEEE section/page**: 17.3.2.4 (p. 2810), 19.3.6 (p. 2879) [P-Deep-1]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None. W3/W5 emphasize disabling Short GI for long range.

### aSymbolTime
- **Source agents**: P-Deep-1, W1
- **Layer**: PHY
- **Criticality**: Mandatory
- **Default value**: 4 µs (Long GI), 3.6 µs (Short GI) [P-Deep-1, W1]
- **Long-distance value**: 4 µs (Long GI preferred) [P-Deep-1, W1]
- **Description**: Total duration of one OFDM symbol including the guard interval.
- **Usage context**: Fundamental unit of data transmission in OFDM/HT.
- **IEEE section/page**: 17.3.2.4 (p. 2810), 19.3.6 (p. 2879) [P-Deep-1]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### aSignalExtension
- **Source agents**: P-Deep-1
- **Layer**: PHY
- **Criticality**: Mandatory (for certain formats)
- **Default value**: 6 µs [P-Deep-1]
- **Long-distance value**: 6 µs [P-Deep-1]
- **Description**: A period of no transmission added after the end of the last symbol of an OFDM frame.
- **Usage context**: Used in ERP-OFDM and HT (Mixed and Greenfield) formats to ensure legacy STAs have enough time for processing.
- **IEEE section/page**: 10.3.8 (p. 1682), 19.3.2 (p. 2873) [P-Deep-1]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### dot11AckTimeout
- **Source agents**: P-Deep-2, W1, W2, W3, W5, K1
- **Layer**: MAC
- **Criticality**: Mandatory
- **Default value**: aSIFSTime + aSlotTime + aRxPHYStartDelay [P-Deep-2]; ~45-75 µs [K1, W2]
- **Long-distance value**: (2 * Propagation Delay) + SIFS + AckTxTime + Buffer [W1, W3, W5, K1]
- **Description**: The interval a STA waits for an ACK frame after transmitting an MPDU.
- **Usage context**: Frame acknowledgment procedure; critical for reliable delivery.
- **IEEE section/page**: 10.3.2.11 [P-Deep-2]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None. W3 notes hardware caps may limit this (e.g. ~800 µs).

### dot11CTSTimeout
- **Source agents**: P-Deep-2, W2, K1
- **Layer**: MAC
- **Criticality**: Mandatory
- **Default value**: aSIFSTime + aSlotTime + aRxPHYStartDelay [P-Deep-2]
- **Long-distance value**: (2 * Propagation Delay) + SIFS + CTSTxTime + Buffer [W2, K1]
- **Description**: The interval a STA waits for a CTS frame after transmitting an RTS frame.
- **Usage context**: RTS/CTS handshake for collision avoidance and hidden node mitigation.
- **IEEE section/page**: 10.3.2.9 [P-Deep-2]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### dot11RTSThreshold
- **Source agents**: P-Deep-2, W2
- **Layer**: MAC
- **Criticality**: Optimization
- **Default value**: 2347 [P-Deep-2, W2]
- **Long-distance value**: Lower values (e.g., 512-1024) [P-Deep-2, W2]
- **Description**: Determines whether an RTS/CTS exchange precedes frame transmission.
- **Usage context**: Collision avoidance; hidden node problem mitigation.
- **IEEE section/page**: Annex C [P-Deep-2]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### dot11FragmentationThreshold
- **Source agents**: P-Deep-2, W2, W3
- **Layer**: MAC
- **Criticality**: Optimization
- **Default value**: 2346 [P-Deep-2, W2, W3]
- **Long-distance value**: Lower values (e.g., 512-1500) [W2, W3]
- **Description**: Maximum size of MPDU before fragmentation occurs.
- **Usage context**: Frame partitioning to improve reliability in noisy/unstable environments.
- **IEEE section/page**: Annex C [P-Deep-2]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### dot11CoverageClass
- **Source agents**: P-Deep-2, W2, K1
- **Layer**: MAC Management
- **Criticality**: Mandatory
- **Default value**: 0 [P-Deep-2, W2, K1]
- **Long-distance value**: Integer = Ceil(Distance_in_km / 0.450) [W2, K1]
- **Description**: Characterizes the BSS radius and adjusts timing parameters (Slot Time and ACK/CTS timeouts).
- **Usage context**: MAC timing adjustment for outdoor deployments.
- **IEEE section/page**: Annex C [P-Deep-2]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### dot11BeaconPeriod
- **Source agents**: P-Deep-3, W4, K2
- **Layer**: MAC Management
- **Criticality**: Optimization
- **Default value**: 100 TU (102.4 ms) [P-Deep-3, W4, K2]
- **Long-distance value**: 200-1000 TU [P-Deep-3, W4, K2]
- **Description**: Time between successive beacon transmissions.
- **Usage context**: BSS synchronization and discovery.
- **IEEE section/page**: 9.4.1.3 (p. 868) [P-Deep-3]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### dot11DTIMPeriod
- **Source agents**: P-Deep-3, W4, K2
- **Layer**: MAC Management
- **Criticality**: Optimization
- **Default value**: 1 to 3 beacon intervals [P-Deep-3, W4, K2]
- **Long-distance value**: 1 [W4, K2]
- **Description**: Number of beacon intervals between successive DTIMs.
- **Usage context**: Broadcast/multicast traffic delivery for power-saving STAs.
- **IEEE section/page**: 9.4.2.14 (p. 951) [P-Deep-3]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### dot11ProbeDelay
- **Source agents**: P-Deep-3
- **Layer**: MAC Management
- **Criticality**: Optimization
- **Default value**: TBD [P-Deep-3]
- **Long-distance value**: TBD [P-Deep-3]
- **Description**: Time a STA waits before transmitting a Probe Request after channel switch.
- **Usage context**: Active scanning.
- **IEEE section/page**: Annex C [P-Deep-3]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### dot11HTAMSDULimit
- **Source agents**: P-Deep-3, W4, K2
- **Layer**: Aggregation
- **Criticality**: Optimization
- **Default value**: 3839 or 7935 octets [P-Deep-3, W4]
- **Long-distance value**: Lower values or Disabled [P-Deep-3, W4, K2]
- **Description**: Maximum size for Aggregate MAC Service Data Unit.
- **Usage context**: Throughput optimization.
- **IEEE section/page**: 9.3.2.2 [P-Deep-3]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### dot11HTAMPDUThreshold
- **Source agents**: P-Deep-3, W4, K2
- **Layer**: Aggregation
- **Criticality**: Optimization
- **Default value**: 65,535 octets [P-Deep-3, W4, K2]
- **Long-distance value**: Optimized for link stability (often 32-64 MPDUs) [W4, K2]
- **Description**: Maximum size for Aggregate MAC Protocol Data Unit.
- **Usage context**: Throughput optimization via subframe aggregation.
- **IEEE section/page**: 9.3.2 [P-Deep-3]
- **Has Phase 1B page ref**: YES
- **Conflicts**: None.

### dot11HTBlockAckWindowSize
- **Source agents**: P-Deep-3, W4, K2
- **Layer**: Aggregation
- **Criticality**: Optimization
- **Default value**: 64 MPDUs [P-Deep-3, W4, K2]
- **Long-distance value**: 64 (Maximize) [P-Deep-3, W4, K2]
- **Description**: Number of frames that can be sent before an acknowledgment is required.
- **Usage context**: High-throughput transmission with selective ACK.
- **IEEE section/page**: TBD [P-Deep-3]
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11ADDBATimeout
- **Source agents**: P-Deep-3, W4, K2
- **Layer**: Aggregation
- **Criticality**: Optimization
- **Default value**: Negotiated [P-Deep-3]; 0 (Disabled) [K2]
- **Long-distance value**: Increased or Disabled (0) [P-Deep-3, W4, K2]
- **Description**: Duration of inactivity before a Block ACK agreement is terminated.
- **Usage context**: Block ACK session management.
- **IEEE section/page**: TBD [P-Deep-3]
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11NAVLimit
- **Source agents**: W2
- **Layer**: MAC
- **Criticality**: TBD
- **Default value**: 32,767 µs [W2]
- **Long-distance value**: Calculated [W2]
- **Description**: Virtual carrier-sensing mechanism timer.
- **Usage context**: Prevents other STAs from transmitting for a set duration.
- **IEEE section/page**: TBD
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11HTMCSIndex
- **Source agents**: W5, K3
- **Layer**: MIMO-MCS
- **Criticality**: Mandatory
- **Default value**: Auto [W5, K3]
- **Long-distance value**: Fixed lower MCS (e.g., MCS 0-11) [W5, K3]
- **Description**: Index defining modulation, coding rate, and spatial streams for HT.
- **Usage context**: Defines PHY data rate.
- **IEEE section/page**: 1.2.840.10036.4.17 (MIB) [W5]
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11HTNss
- **Source agents**: W5, K3
- **Layer**: MIMO-MCS
- **Criticality**: Optimization
- **Default value**: Max supported by hardware [W5]
- **Long-distance value**: 1 or 2 (Reduced to 1 for extreme range) [W5, K3]
- **Description**: The number of independent data streams sent simultaneously using MIMO.
- **Usage context**: Multiplexing gain vs diversity gain.
- **IEEE section/page**: 1.2.840.10036.4.15.1.6 (MIB) [W5]
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11MaxTransmitPower
- **Source agents**: W5, K3
- **Layer**: Regulatory
- **Criticality**: Mandatory
- **Default value**: 20–30 dBm [W5]
- **Long-distance value**: Max legal/hardware (e.g., 30 dBm) [W5, K3]
- **Description**: Maximum power level the radio is allowed to transmit.
- **Usage context**: Regulatory compliance and link budget.
- **IEEE section/page**: 1.2.840.10036.4.3.1.2 (MIB) [W5]
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11EIRPLimit
- **Source agents**: W5, K3
- **Layer**: Regulatory
- **Criticality**: Mandatory
- **Default value**: 36 dBm [W5, K3]
- **Long-distance value**: PtP max (e.g., 53 dBm or No Limit in US) [W5, K3]
- **Description**: Equivalent Isotropically Radiated Power.
- **Usage context**: Regulatory limit for total radiated power.
- **IEEE section/page**: N/A
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11AntennaGain
- **Source agents**: W5, K3
- **Layer**: Regulatory
- **Criticality**: Optimization
- **Default value**: 2–5 dBi [W5, K3]
- **Long-distance value**: 20–34 dBi [W5, K3]
- **Description**: Directional gain of the antenna.
- **Usage context**: Overcoming Free Space Path Loss.
- **IEEE section/page**: N/A
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11HTSTBC
- **Source agents**: W5, K3
- **Layer**: MIMO-MCS
- **Criticality**: Optimization
- **Default value**: Disabled or Auto [W5]
- **Long-distance value**: Enabled [W5, K3]
- **Description**: Technique to transmit multiple copies of a data stream across several antennas.
- **Usage context**: Improving reliability when multiplexing gain is low.
- **IEEE section/page**: TBD
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### aPLCPPreambleDuration
- **Source agents**: W1, K1
- **Layer**: PHY
- **Criticality**: Mandatory
- **Default value**: 36 µs (HT-Mixed) [W1]
- **Long-distance value**: Default (remains constant) [W1, K1]
- **Description**: Initial part of a frame for synchronization.
- **Usage context**: Synchronization and training.
- **IEEE section/page**: Section 19.3.3 [W1]
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11AssociationResponseTimeOut
- **Source agents**: W4
- **Layer**: MAC Management
- **Criticality**: Mandatory
- **Default value**: 512 TU [W4]
- **Long-distance value**: 1000-2048 TU [W4]
- **Description**: Time a STA waits for an Association Response.
- **Usage context**: Initial connection handshake.
- **IEEE section/page**: TBD
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11AuthenticationResponseTimeOut
- **Source agents**: W4
- **Layer**: MAC Management
- **Criticality**: Mandatory
- **Default value**: 512 TU [W4]
- **Long-distance value**: 1000 TU [W4]
- **Description**: Time a STA waits for the next frame in authentication sequence.
- **Usage context**: Initial 802.11 authentication phase.
- **IEEE section/page**: TBD
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11MinChannelTime
- **Source agents**: W4
- **Layer**: MAC Management
- **Criticality**: Optimization
- **Default value**: 10-20 ms [W4]
- **Long-distance value**: 20-40 ms [W4]
- **Description**: Min time spent on channel during active scan.
- **Usage context**: Active scanning.
- **IEEE section/page**: TBD
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11MaxChannelTime
- **Source agents**: W4
- **Layer**: MAC Management
- **Criticality**: Optimization
- **Default value**: 30-60 ms [W4]
- **Long-distance value**: 60-100 ms [W4]
- **Description**: Max time spent on channel during active scan.
- **Usage context**: Active scanning.
- **IEEE section/page**: TBD
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11HTChannelWidth
- **Source agents**: W3, W5, K3
- **Layer**: PHY
- **Criticality**: Optimization / Mandatory (for range)
- **Default value**: 20 MHz [W5, K3]
- **Long-distance value**: 20 MHz (preferred for range) [W3, W5, K3]
- **Description**: Spectral bandwidth used for transmission.
- **Usage context**: 802.11n bonds two 20 MHz channels for 40 MHz.
- **IEEE section/page**: 1.2.840.10036.4.15 (MIB) [W5]
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### aPIFSTime
- **Source agents**: K1
- **Layer**: MAC
- **Criticality**: Optimization
- **Default value**: SIFS + 1 * SlotTime [K1]
- **Long-distance value**: SIFS + (Adjusted SlotTime) [K1]
- **Description**: Intermediate inter-frame space used by AP for priority access.
- **Usage context**: Used in PCF or for Beacon transmission.
- **IEEE section/page**: TBD
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11ATIMWindow
- **Source agents**: K2
- **Layer**: MAC Management
- **Criticality**: Optimization (IBSS only)
- **Default value**: 0 TUs [K2]
- **Long-distance value**: 5-10 TUs [K2]
- **Description**: Announcement Traffic Indication Message window.
- **Usage context**: IBSS power management window.
- **IEEE section/page**: TBD
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.

### dot11ProbeResponseTimeout
- **Source agents**: K2
- **Layer**: MAC Management
- **Criticality**: Optimization
- **Default value**: 10-30 ms [K2]
- **Long-distance value**: 50-100 ms [K2]
- **Description**: Duration a STA waits for response after probe request.
- **Usage context**: Active scanning discovery.
- **IEEE section/page**: TBD
- **Has Phase 1B page ref**: NO
- **Conflicts**: None.
