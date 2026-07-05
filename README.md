# URLGuard

**On-device ML-powered phishing defense with privacy-first P2P threat sharing and homograph detection.**

[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Version](https://img.shields.io/badge/Version-1.0.0-blue.svg)](https://github.com/AdityaP700/privacyguard)
[![Hackathon](https://img.shields.io/badge/Amplicode%20Hackathon-13th%20%2F%20925%20Submissions-gold.svg)](https://devpost.com/software/privacyguard-zh9r3n)
[![Track](https://img.shields.io/badge/Social%20Security%20%26%20Privacy%20Track-2nd%20Place-silver.svg)](https://devpost.com/software/privacyguard-zh9r3n)

[Demo](https://youtu.be/GnMNnVqbZy0) · [Devpost](https://devpost.com/software/privacyguard-zh9r3n) · [Report an Issue](https://github.com/AdityaP700/PrivacyGuard/issues)

---

## Recognition

Ranked **13th out of 925 submissions** at the Amplicode Hackathon and **2nd Place in the Social Security & Privacy track**. PrivacyGuard was evaluated against commercial-grade browser security tools and research prototypes from participants across the world.

---

## Overview

URLGuard is a Chrome extension that protects users from phishing attacks and malicious websites using a combination of on-device machine learning, heuristic analysis, and Unicode homograph detection. Built with a privacy-first architecture — all URL analysis happens locally in the browser; no data is transmitted to any external server.

---

## Features

### AI-Powered Detection
- Custom TensorFlow.js model trained on 100k+ balanced samples, achieving 88.88% test accuracy
- 16 lexical URL features extracted and analyzed entirely on-device
- Sub-500ms inference latency per URL

### Tiered Alert System
| Risk Level | Score Range | Behavior |
|---|---|---|
| High Risk | > 75 | Full-page interstitial block |
| Caution | 30 – 75 | Non-intrusive corner notification |
| Safe | < 30 | Silent background monitoring |

### Multi-Layer Protection
- **Homograph detection** — flags Punycode (`xn--`) and mixed-script Unicode spoofing
- **Heuristic engine** — evaluates HTTPS presence, suspicious form elements, IP-based domains, URL entropy
- **Smart whitelisting** — user-controlled trust preferences persist via local storage
- **P2P threat sharing** — mock implementation demonstrating a community-driven, privacy-preserving threat network

---

## Architecture

```
PrivacyGuard/
├── js/
│   ├── content.js          # Core analysis engine and alert injection
│   ├── tf.min.js           # TensorFlow.js runtime
│   └── tfjs_model/         # Converted TF.js Graph Model
├── popup/
│   ├── popup.html          # Extension popup interface
│   ├── popup.js            # UI logic and controls
│   └── popup.css           # Popup styling
├── manifest.json           # Extension configuration (Manifest V2)
└── icons/
```

### Detection Pipeline

```
Browser Extension
├── Content Script        ML inference, heuristics, homograph checks, DOM alerts
├── Popup Interface       Risk score display, user controls, per-URL analytics
└── Background Service    Persistent storage, settings management, P2P simulation

Detection Engines
├── ML Engine             TF.js model, 16 lexical features, sigmoid output
├── Heuristic Analysis    HTTPS, form detection, URL structure patterns
└── Homograph Detection   Punycode decoding, mixed-script analysis, confusable mapping
```

### Alert Isolation via Shadow DOM

Alerts are injected using a closed Shadow DOM to prevent CSS bleed into host page layouts:

```javascript
const createAlert = (riskData) => {
    const container = document.createElement('div');
    const shadow = container.attachShadow({ mode: 'closed' });
    shadow.innerHTML = `
        <style>
            .privacy-guard-alert {
                position: fixed;
                z-index: 2147483647;
                font-family: -apple-system, BlinkMacSystemFont, sans-serif;
            }
        </style>
        ${getAlertHTML(riskData)}
    `;
    document.body.appendChild(container);
};
```

---

## Model Details

### Training

| Property | Value |
|---|---|
| Primary dataset | [ebubekirbbr/dephides](https://github.com/ebubekirbbr/dephides) (~100k samples) |
| Validation dataset | [Tranco Top 1M](https://tranco-list.eu/) |
| Model accuracy | 88.88% |
| Architecture | `Input(16) → Dense(32, ReLU) → Dense(16, ReLU) → Dense(1, Sigmoid)` |
| Conversion pipeline | Keras → TensorFlow SavedModel → TF.js Graph Model (tensorflowjs_converter v4.22.0) |
| Feature normalization | MinMaxScaler parameters ported to JavaScript |

### Feature Set (16 Lexical Features)

`url_length`, `hostname_length`, `path_length`, `query_length`, `num_dots`, `num_hyphens`, `num_at`, `num_question_marks`, `num_equals`, `num_underscore`, `num_percent`, `num_slash`, `has_https`, `has_ip`, `num_digits`, `num_letters`

### Runtime Performance

| Metric | Value |
|---|---|
| Analysis latency | ~450ms per URL |
| Additional memory | ~15MB |
| CPU impact | < 2% during analysis |
| Model load time | 1.8s (cached after first load) |

---

## Installation

```bash
git clone https://github.com/AdityaP700/PrivacyGuard.git
cd PrivacyGuard
```

1. Open `chrome://extensions/`
2. Enable **Developer Mode** (top-right toggle)
3. Click **Load unpacked** and select the `PrivacyGuard` directory
4. Pin the extension for quick access

### Test Scenarios

```bash
# Serve local test files
python -m http.server 8000

# Green  — safe browsing
https://www.google.com

# Yellow — suspicious HTTP site
http://localhost:8000/college.html

# Red    — Punycode homograph attack (fake PayPal)
http://www.xn--pypal-4ve.com/
```

---

## Developer Reference

```javascript
// Inspect current whitelist
chrome.storage.local.get('privacyGuardWhitelist', console.log);

// View P2P data stores
chrome.storage.local.get(
    ['privacyGuardP2PUserPhishing', 'privacyGuardP2PUserSafe'],
    console.log
);

// Trigger manual analysis
analyzeCurrentURL().then(console.log);
```

---

## Known Limitations

- P2P threat sharing is a mock implementation backed by `chrome.storage.local`; no real peer network exists
- Homograph detection covers Latin-script confusables; full multilingual coverage is pending
- False positive rate is approximately 5.8%, addressable via per-domain whitelisting
- Page content analysis is limited to form element detection; no DOM-level behavioral analysis
- Optimized for Chromium-based browsers (Chrome, Edge); Firefox requires API polyfills

---

## Roadmap

### v1.1
- Retrain model on a larger, more diverse multilingual dataset
- Dark mode UI support
- Threat statistics dashboard with trend visualization
- Export and import of user settings and whitelists

### Future Work
- **Federated learning** — collaborative model improvement without raw data collection
- **Real P2P via WebRTC** — decentralized, anonymized threat sharing with trust scoring
- **Visual identity checks** — perceptual hashing for logo and favicon mismatch detection
- **Domain intelligence** — Newly Registered Domain (NRD) detection and WHOIS integration
- **Manifest V3 migration** — `declarativeNetRequest` for improved performance and security
- **Cross-browser support** — Firefox (via `browser.*` polyfills), Safari (WebExtensions API)

---

## Contributing

```bash
# Fork and clone
git clone https://github.com/AdityaP700/PrivacyGuard.git

# Create a feature branch
git checkout -b feature/your-feature-name

# Test changes thoroughly before submitting a pull request
```

Bug reports and feature requests: [GitHub Issues](https://github.com/AdityaP700/PrivacyGuard/issues)  
Discussions: [GitHub Discussions](https://github.com/AdityaP700/PrivacyGuard/discussions)

---

## Acknowledgments

- Training data: [ebubekirbbr/dephides](https://github.com/ebubekirbbr/dephides), [Tranco](https://tranco-list.eu/)
- ML runtime: [TensorFlow.js](https://www.tensorflow.org/js)
- Hackathon: [Amplicode](https://devpost.com/software/privacyguard-zh9r3n)

---

## License

MIT — see [LICENSE](LICENSE) for details.

**Contact:** adityaa32078@gmail.com · [@AdityaPat_](https://x.com/AdityaPat_) · [LinkedIn](https://linkedin.com/in/yourprofile)
