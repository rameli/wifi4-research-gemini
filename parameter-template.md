# Parameter Metadata Template

Use this template for every 802.11n parameter documented during any research phase. Copy and fill in all fields. Use "N/A" for fields that do not apply. Use "TBD" for fields not yet determined.

---

## [Parameter Name]

### Identification

- **Official IEEE name**: [Exact name as it appears in IEEE 802.11-2020, e.g., `aSlotTime`, `dot11RTSThreshold`]
- **Common name(s)**: [Other names used in practice, drivers, or literature]
- **Layer**: [PHY / MAC / MAC management / Aggregation / MIMO-MCS / Regulatory]
- **Criticality**: [Mandatory / Optimization]
  - Mandatory = link will not establish or function without modification
  - Optimization = improves performance or reliability but not strictly required

### Description

- **Official IEEE description**: [Description as found in IEEE 802.11-2020 standard — quote or close paraphrase]
- **Functional summary**: [1-2 sentence plain-language explanation of what this parameter controls]

### Usage context

- **How it is used**: [Brief explanation of when and how this parameter is invoked during WiFi operation, e.g., "Used by the MAC to determine how long to wait for an ACK frame after transmitting a data frame"]
- **Scenarios**: [Specific scenarios where this parameter matters, e.g., "Every unicast data frame exchange", "Only during RTS/CTS handshake", "During scanning for networks"]

### Values

- **Default value**: [Standard-defined default value with units]
- **Long-distance value**: [Recommended or calculated value for long-distance operation, with formula if applicable]
- **Value range**: [Acceptable range, if defined in the standard]

### Distance dependency

- **Why it matters for long distance**: [How propagation delay or distance affects this parameter at scales up to hundreds of km]
- **Calculation/formula**: [If the parameter value depends on distance, provide the formula, e.g., `ACKTimeout = SIFS + SlotTime + propagation_delay_round_trip`]

### Channel width notes

- **20 MHz behavior**: [Value or behavior at 20 MHz, or "Same as default"]
- **40 MHz behavior**: [Value or behavior at 40 MHz, or "Same as 20 MHz"]
- **Differences**: [Any notable differences between channel widths]

### MIMO/MCS relevance

- **Interaction with spatial streams**: [How this parameter interacts with MIMO spatial streams, or "None"]
- **MCS selection impact**: [How this parameter affects or is affected by MCS rate selection, or "None"]

### Regulatory notes

- **EIRP/power constraints**: [Relevant regulatory constraints, or "None"]
- **Band-specific considerations**: [2.4 GHz vs 5 GHz differences, or "None"]

### IEEE 802.11-2020 standard reference

- **Section**: [Section number in the standard, e.g., "9.4.2.28"]
- **Page(s)**: [Page number(s) in `sources/802.11-2020-reissue-2022-01.pdf`]
- **Table/Figure**: [Reference to specific tables or figures, if applicable]

### Related parameters

- **Dependencies**: [Parameters this one depends on, e.g., "Depends on SIFS and SlotTime"]
- **Dependents**: [Parameters that depend on this one]
- **Interactions**: [Other parameters that should be adjusted in coordination]

### Sources

- **Web sources**: [URLs where this parameter was discussed in context of long-distance WiFi]
- **Book reference**: [Page/chapter in O'Reilly "802.11 Wireless Networks: The Definitive Guide" if applicable]
- **Training knowledge**: [Note if this information came from model training knowledge]
