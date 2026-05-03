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
| 0 | `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266` | Admin (tous les rôles) | `0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80` |
| 2 | `0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC` | BIO_ENGINEER | `0x5de4111afa1a4b94908f83103eb1f1706367c2e68ca870fc3fb9a804cdab365a` |
| 3 | `0x90F79bf6EB2c4f870365E785982E1f101E93b906` | RADIOLOGIST | `0x7c852118294e51e653712a81e05800f419141751be58f605c371e15141b007a6` |
| 4 | `0x15d34AAf54267DB7D7c367839AAf71A00a2C6A65` | TECHNICIAN | `0x47e179ec197488593b187f80a00eb0da91f1b9d0b13f8733639f19c30a34926a` |

---

## Flux global

```
BIO_ENGINEER          RADIOLOGIST          TECHNICIAN          BIO_ENGINEER
     │                     │                    │                    │
 [Étape 1]                 │                    │                    │
 Mint NFT (#1)             │                    │                    │
     │                     │                    │                    │
 [Étape 2]                 │                    │                    │
 Register Firmware         │                    │                    │
     │                     │                    │                    │
     │               [Étape 3]                  │                    │
     │               Create Ticket (#1)         │                    │
     │                     │                    │                    │
 [Étape 4]                 │                    │                    │
 Assign Technician ─────────────────────────────►                    │
     │                     │                    │                    │
     │                     │              [Étape 5]                  │
     │                     │              Start + Submit Report      │
     │                     │                    │                    │
     │                     │                    │              [Étape 6]
     │                     │                    │              Validate ✅
```

---

## Étape 1 — Mint Equipment NFT

**Compte :** Account #2 — BIO_ENGINEER  
**Page :** `/equipment` → bouton **"Mint Equipment"**

| Champ | Valeur |
|-------|--------|
| Model | `Siemens MAGNETOM Aera` |
| Serial Number | `SN-2024-IRM-001` |
| Location | `CHU Alger - Service Radiologie` |

**Action :** Cliquer **"Mint Equipment"** → confirmer dans MetaMask  
**Résultat :** Token NFT créé avec **ID = 1** ✅

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

**Compte :** Account #3 — RADIOLOGIST → switcher MetaMask  
**Page :** `/tickets` → formulaire **"Step 1: Declare Equipment Fault"**

| Champ | Valeur |
|-------|--------|
| Equipment Token ID | `1` |
| Fault Documentation | importer un fichier PDF ou image quelconque |

**Action :** Cliquer **"Create Ticket"** → confirmer MetaMask  
**Résultat :** Ticket créé avec **ID = 1**, statut `OPEN` ✅

---

## Étape 4 — Assigner le technicien

**Compte :** Account #2 — BIO_ENGINEER → switcher MetaMask  
**Page :** `/tickets` → page **"Gestion des Pannes"** → onglet `Ouverts`

| Champ | Valeur |
|-------|--------|
| Ticket | #1 → cliquer **"Assigner"** |
| Technicien | sélectionner `0x15d34AAf54267DB7D7c367839AAf71A00a2C6A65` (Account #4) |

**Action :** Cliquer **"Assigner"** → confirmer MetaMask  
**Résultat :** Ticket #1 passe en statut `ASSIGNED` ✅  
> Le contrat vérifie automatiquement que le technicien possède un SBT valide.

---

## Étape 5 — Soumettre le rapport d'intervention

**Compte :** Account #4 — TECHNICIAN → switcher MetaMask  
**Page :** `/tickets` → page **"Mes Interventions"**

| Action | Bouton |
|--------|--------|
| 1. Démarrer l'intervention | **"Démarrer"** → confirmer MetaMask (statut → `IN_PROGRESS`) |
| 2. Sélectionner le rapport | **"📎 Rapport"** → choisir un fichier PDF |
| 3. Soumettre | **"Signer & Soumettre"** → confirmer MetaMask |

**Résultat :** Ticket #1 passe en statut `PENDING_VALIDATION` ✅  
Le rapport est uploadé sur IPFS et le CID est enregistré on-chain.

---

## Étape 6 — Valider l'intervention

**Compte :** Account #2 — BIO_ENGINEER → switcher MetaMask  
**Page :** `/tickets` → onglet `À valider`

| Action | Description |
|--------|-------------|
| Vérifier les signatures | Les 2 icônes (Technicien + Bio-Ingénieur) doivent être visibles |
| Cliquer **"✅ Valider"** | Confirmer dans MetaMask |

**Résultat :** Ticket #1 passe en statut `CLOSED` ✅

> ⚠️ **Note environnement local :** La validation appelle `PaymentEscrow.release()` qui nécessite
> un dépôt USDC préalable. En local, le USDC est un contrat mock (`0x1`) donc cette transaction
> échoue. Tout le reste du flux fonctionne normalement. Pour tester la validation complète,
> un MockERC20 doit être déployé et le dépôt effectué avant la validation.

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
**Page :** `/certifications` → panneau gauche **"Mint New Certification SBT"**

| Champ | Valeur |
|-------|--------|
| Technician Wallet Address | `0x15d34AAf54267DB7D7c367839AAf71A00a2C6A65` |
| Manufacturer | `Siemens` |
| Specialization | `MRI` |
| Expiry Date | `2030-01-01` |

**Action :** Cliquer **"Mint SBT"** → confirmer MetaMask  
**Résultat :** SBT non-transférable créé dans le wallet du technicien ✅

### Révoquer un SBT

| Champ | Valeur |
|-------|--------|
| SBT Token ID to Revoke | `1` |

**Action :** Cliquer **"Revoke SBT"** → confirmer MetaMask  
**Résultat :** Le technicien ne peut plus être assigné à un ticket ✅

---

## Adresses des smart contracts (localhost)

| Contrat | Adresse |
|---------|---------|
| BioGovAccessControl | `0x5FbDB2315678afecb367f032d93F642f64180aa3` |
| EquipmentNFT | `0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512` |
| CertificationSBT | `0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0` |
| FirmwareProofRegistry | `0xCf7Ed3AccA5a467e9e704C703E8D87F634fB0Fc9` |
| PaymentEscrow | `0xDc64a140Aa3E981100a9becA4E685f962f0cF6C9` |
| MaintenanceController | `0x5FC8d32690cc91D4c39d9d3abcBD16989F875707` |

---

## Commandes pour relancer l'application

```bash
# Terminal 1 — Blockchain
cd "blockchain"
npx hardhat node

# Terminal 2 — Déploiement (une seule fois après lancement du node)
cd "blockchain"
npx hardhat run scripts/deploy.ts --network localhost
npx hardhat run scripts/assign-roles.ts --network localhost

# Terminal 3 — Backend
cd "backend"
mvn spring-boot:run

# Terminal 4 — Frontend
cd "frontend"
npm start
```

---

*Scénario généré le 03 mai 2026 — Bio-Gov Digital Twins v1.0*
