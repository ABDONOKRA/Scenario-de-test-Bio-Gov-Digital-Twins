# Scénario de test — Bio-Gov Digital Twins

> **Application :** Bio-Gov — Gestion décentralisée des équipements médicaux  
> **URL Frontend :** http://localhost:3000  
> **URL Backend :** http://localhost:8080/api  
> **URL Blockchain :** http://127.0.0.1:8545  

---

## Comptes MetaMask à importer

Réseau : **Hardhat Local** — chainId `31337` — RPC `http://127.0.0.1:8545`

| # | Adresse | Rôle | Clé privée |
|---|---------|------|-----------|
| 0 | `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266` | DEFAULT_ADMIN_ROLE | `0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80` |
| 2 | `0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC` | BIO_ENGINEER_ROLE | `0x5de4111afa1a4b94908f83103eb1f1706367c2e68ca870fc3fb9a804cdab365a` |
| 3 | `0x90F79bf6EB2c4f870365E785982E1f101E93b906` | RADIOLOGIST_ROLE | `0x7c852118294e51e653712a81e05800f419141751be58f605c371e15141b007a6` |
| 4 | `0x15d34AAf54267DB7D7c367839AAf71A00a2C6A65` | TECHNICIAN_ROLE | `0x47e179ec197488593b187f80a00eb0da91f1b9d0b13f8733639f19c30a34926a` |
| 5 | `0x9965507D1a55bcC2695C58ba16FB37d819B0A4dc` | TECHNICIAN_ROLE (2) | `0x8b3a350cf5c34c9194ca85829a2df0ec3153be0318b5e2d3348e872092edffba` |
| 6 | `0x976EA74026E726554dB657fA54763abd0C3a0aa9` | Admin dédié (optionnel) | `0x92db14e403b83dfe3df233f83dfa3a0d7096f21ca9b0d6d6b8d88b2b4ec1564e` |

---

## Adresses des smart contracts (déploiement actuel)

| Contrat | Adresse |
|---------|---------|
| BioGovAccessControl | `0x610178dA211FEF7D417bC0e6FeD39F05609AD788` |
| EquipmentNFT | `0xB7f8BC63BbcaD18155201308C8f3540b07f84F5e` |
| CertificationSBT | `0xA51c1fc2f0D1a1b8494Ed1FE312d7C3a78Ed91C0` |
| FirmwareProofRegistry | `0x0DCd1Bf9A1b36cE34237eEaFef220932846BCD82` |
| PaymentEscrow | `0x9A676e781A523b5d0C0e43731313A708CB607508` |
| MaintenanceController | `0x0B306BF915C4d645ff596e518fAf3F9669b97016` |

---

## Flux global

```
ADMIN          BIO_ENGINEER        RADIOLOGIST       TECHNICIAN       BIO_ENGINEER
  │                 │                   │                 │                │
[Étape 0]           │                   │                 │                │
Gérer rôles         │                   │                 │                │
  │                 │                   │                 │                │
  │           [Étape 1]                 │                 │                │
  │           Mint NFT (#1)             │                 │                │
  │                 │                   │                 │                │
  │           [Étape 2]                 │                 │                │
  │           Register Firmware         │                 │                │
  │                 │                   │                 │                │
  │                 │             [Étape 3]               │                │
  │                 │             Create Ticket           │                │
  │                 │                   │                 │                │
  │           [Étape 4]                 │                 │                │
  │           Assign Technician ─────────────────────────►                │
  │                 │                   │                 │                │
  │                 │                   │           [Étape 5]              │
  │                 │                   │           Start + Submit         │
  │                 │                   │                 │                │
  │                 │                   │                 │          [Étape 6]
  │                 │                   │                 │          Validate ✅
```

---

## Étape 0 — Gestion des Rôles (Admin)

**Compte :** Account #0 — Admin (`0xf39Fd6e...`)  
**Page :** `/admin` → **"Gestion des Rôles"**

Permet d'accorder ou révoquer les rôles `TECHNICIAN_ROLE`, `BIO_ENGINEER_ROLE`, `RADIOLOGIST_ROLE` à n'importe quelle adresse wallet.

> Les rôles sont lus directement depuis la blockchain — tout wallet enregistré on-chain reçoit automatiquement ses rôles au login.

---

## Étape 1 — Mint Equipment NFT

**Compte :** Account #2 — BIO_ENGINEER  
**Page :** `/equipment` → bouton **"Mint Equipment"**

| Champ | Valeur |
|-------|--------|
| Model | `Siemens MAGNETOM Aera` |
| Serial Number | `SN-2024-IRM-001` |
| Location | `CHU AGADIR - Service Radiologie` |

**Action :** Cliquer **"Mint Equipment"** → confirmer dans MetaMask  
**Résultat :** Token NFT créé avec **ID = 1**, statut `Actif` ✅

---

## Étape 2 — Enregistrer le Firmware officiel

**Compte :** Account #2 — BIO_ENGINEER  
**Page :** `/firmware` → panneau gauche **"Register Official Firmware Hash"**

| Champ | Valeur |
|-------|--------|
| Equipment Token ID | `1` |
| Firmware Version | `3.2.1` |
| Firmware Hash | `firmware_v321_siemens_official` |

**Action :** Cliquer **"Register Firmware"** → confirmer MetaMask  
**Résultat :** Hash enregistré on-chain de façon immuable ✅

---

## Étape 3 — Déclarer une panne

**Compte :** Account #3 — RADIOLOGIST  
**Page :** `/tickets` → formulaire de déclaration

| Champ | Valeur |
|-------|--------|
| Equipment Token ID | `1` |
| Fault Documentation | importer un fichier PDF ou image |

**Action :** Cliquer **"Create Ticket"** → confirmer MetaMask  
**Résultat :** Ticket créé, statut `OPEN`, équipement passe en `En maintenance` ✅

---

## Étape 4 — Assigner le technicien

**Compte :** Account #2 — BIO_ENGINEER  
**Page :** `/tickets` → **"Gestion des Pannes"** → onglet `Ouverts`

| Champ | Valeur |
|-------|--------|
| Ticket | #1 → cliquer **"Assigner"** |
| Technicien | sélectionner dans la liste (seuls les techniciens certifiés apparaissent) |

**Action :** Cliquer **"Assigner"** → confirmer MetaMask  
**Résultat :** Ticket #1 passe en statut `ASSIGNED` ✅

> Le contrat vérifie automatiquement que le technicien possède un SBT valide.

---

## Étape 5 — Soumettre le rapport d'intervention

**Compte :** Account #4 — TECHNICIAN  
**Page :** `/tickets` → **"Mes Interventions"**

| Action | Bouton |
|--------|--------|
| 1. Démarrer l'intervention | **"Démarrer"** → confirmer MetaMask (statut → `IN_PROGRESS`) |
| 2. Sélectionner le rapport | **"📎 Rapport"** → choisir un fichier PDF |
| 3. Soumettre | **"Signer & Soumettre"** → confirmer MetaMask |

**Résultat :** Ticket #1 passe en statut `PENDING_VALIDATION` ✅  
Le rapport est uploadé sur IPFS (Pinata) et le CID est enregistré on-chain.

---

## Étape 6 — Valider l'intervention

**Compte :** Account #2 — BIO_ENGINEER  
**Page :** `/tickets` → onglet `À valider`

| Action | Description |
|--------|-------------|
| Vérifier les signatures | L'icône Technicien doit être verte |
| Cliquer **"✅ Valider"** | Confirmer dans MetaMask |

**Résultat :** Ticket #1 passe en statut `CLOSED`, équipement revient à `Actif` ✅

---

## Étape 7 — Vérifier l'intégrité du Firmware (optionnel)

**Compte :** Account #2 — BIO_ENGINEER  
**Page :** `/firmware` → panneau droit **"Verify Firmware Integrity"**

### Test 1 — Firmware authentique ✅
| Champ | Valeur |
|-------|--------|
| Equipment Token ID | `1` |
| Current Firmware Hash | `firmware_v321_siemens_official` |

**Résultat :** `FIRMWARE MATCHES — Integrity Verified`

### Test 2 — Firmware altéré ❌
| Champ | Valeur |
|-------|--------|
| Equipment Token ID | `1` |
| Current Firmware Hash | `firmware_tampered_v999` |

**Résultat :** `MISMATCH — Possible Tampering!`

---

## Étape 8 — Minter une Certification SBT (optionnel)

**Compte :** Account #2 — BIO_ENGINEER  
**Page :** `/certifications` → **"Émettre un SBT"**

| Champ | Valeur |
|-------|--------|
| Adresse du Technicien | adresse du nouveau technicien |
| Fabricant | `Siemens Healthineers` |
| Spécialisation | `IRM / Scanner` |
| Date d'expiration | sélectionner via le calendrier |

**Action :** Cliquer **"Émettre le SBT"** → confirmer MetaMask  
**Résultat :** SBT non-transférable créé, technicien certifié ✅

---

## Commandes pour relancer l'application

```bash
# Terminal 1 — Blockchain
cd blockchain
npx hardhat node

# Terminal 2 — Déploiement (une seule fois après lancement du node)
cd blockchain
npx hardhat run scripts/deploy.ts --network localhost
npx hardhat run scripts/assign-roles.ts --network localhost

# Terminal 3 — Backend
cd backend
mvn spring-boot:run

# Terminal 4 — Frontend
cd frontend
npm start
```

---

*Mis à jour le 04 mai 2026 — Bio-Gov Digital Twins v1.0*
