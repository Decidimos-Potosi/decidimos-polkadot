# Decidimos on Polkadot

Decidimos is the app through which the people of Potosí, Bolivia, decide how their share of the Uyuni lithium project is spent, and receive it. This repository holds the Polkadot side of Decidimos: the payout rail, the wallet integration and the minors' savings, built by Heitaria Swiss AG.

- Polkadot OpenGov proposal: link follows after submission
- Forum discussion: https://forum.polkadot.network/t/18650
- App: https://apps.apple.com/app/id6809736620 (Android in closed testing)
- Website: https://heitaria.ch/decidimos/

## What it does

The project commits about US$53.9 million a year to Potosí for 43 years of production.

- About US$6.3 million a year is paid every month in USDT to each resident of Río Grande and of the 52 communities of the TIOC Nor Lípez: about 14,000 people.
- About US$47.6 million a year funds works that the communities of the department (861,292 people) propose and vote on in Decidimos.
- When lithium stays above US$20,000/t for 12 months, every registered wallet in the department receives a yearly dividend (Dividendo Potosí).

Signed memoranda (24 October 2026) with CUPCONL and the community of Río Grande, ratified by their assemblies, make Decidimos the mandatory channel for all per-capita payments and all allocations of the community funds. Signature pages: [Río Grande](docs/Memorandum_Rio_Grande_firmas.jpg), [CUPCONL](docs/Memorandum_CUPCONL_firmas.jpg).

## Why we do this

The world needs lithium, but mining has a bad name: for centuries, the wealth under the ground in Potosí left and the people who live on it stayed poor. Even Pope Leo XIV has condemned the "colonization" of minerals by mining companies.

We have been developing this project since 2024 to change that. We call it globalisation 2.0, or social mining: the people who live next to the resource decide how their share is spent and receive it directly, every month, on their phone, with no one in between and in the open.

Uyuni is the first project to ring-fence at least 15% of the private partners' profits for the people of the territory. Decidimos is built to be copied: the next lithium, copper, tin or silver project, in Bolivia or elsewhere, can use the same model and the same payout rail on Polkadot.

## The lithium project

Who pays: the private partners of the project, not the government. They pay the share of Potosí directly: fixed payments plus at least 15% of their profits, paid out through Decidimos.

The plant runs on solar power, off-grid: a 283.6 MW photovoltaic field with 792 MWh of batteries for the electricity, and a 389.1 MW-thermal solar field with molten-salt storage for the heat, day and night. Diesel is only an emergency backup.

Water: the plant consumes no fresh water in operation; it produces water. The condensate from evaporating the brine exceeds the process demand by about 8,000 m³ a day (about 3,800 m³ in the pessimistic case). After the camp, the water committed to the communities and the cleaning of the solar panels, thousands of cubic metres a day remain available. The process uses no evaporation ponds.

Phase 0 desk studies of the plant, made before drilling (not final results):

- Energy Supply Assessment, Phase 0: [English](docs/project/Uyuni_Energy_Supply_Assessment_Phase0_EN_v2.2.pdf) · [Español](docs/project/Uyuni_Evaluacion_Suministro_Energetico_Fase0_ES_v2.2.pdf)
- Hydrological and Hydrogeological Assessment, Phase 0 (for CUPCONL): [English](docs/project/Uyuni_Hydrogeological_Assessment_Phase0_CUPCONL_EN_v1.7.pdf) · [Español](docs/project/Uyuni_Evaluacion_Hidrogeologica_Fase0_CUPCONL_ES_v1.7.pdf)

## Status (November 2026)

- Governance app in closed beta with the 53 communities of the TIOC Nor Lípez, in Spanish, Quechua and Aymara.
- Wallet running on Polkadot Asset Hub mainnet. First real USDT payments made in a shop in Río Grande (October 2026).
- Chanchito savings live: residents lend their own USDT in Hydration's money market.
- Payout contracts deployed on Polkadot Hub TestNet (Paseo). Two internal reviews done; every finding is replayed in the test suite.
- External audit by Hacken in April 2027. The report will be published here.

## Architecture

```
Community rolls (ratified by the assemblies)
        │
        ▼
Decidimos server ── builds the monthly list ── Merkle root
        │
        ▼
Heitaria treasury (2-of-3 multisig) ── funds + publishes the month in one transaction
        │
        ▼
PayoutDistributor (Polkadot Hub) ── 1-day delay: CUPCONL checks, can veto or freeze
        │                         └─ 30% of each minor's share ──► MinorVault (locked until 18)
        ▼
Relayer pays every line (claimMany) ──► resident's wallet on Asset Hub (USDT, asset 1984)
        │
        ▼
Resident pays shops, transfers, saves in the Chanchito (Hydration)
```

### Contracts (Solidity, Polkadot Hub via pallet-revive)

| Contract | Role |
|---|---|
| Parties.sol | Three parties: treasury (Heitaria 2-of-3 multisig), watchdog (CUPCONL), independent arbiter. Replacing a party needs 2 of 3 and 30 public days. The watchdog can freeze both money contracts (30 days, renewable). |
| Governed.sol | Shared base: holds USDT and a minimum balance. The treasury can only withdraw unreserved surplus; unused month balances unlock only after 90 days and with CUPCONL's approval. |
| PayoutDistributor.sol | Monthly payouts by Merkle root. Published amounts are reserved. One-day claim delay for CUPCONL to check the list; CUPCONL can veto a month. No line can be paid twice. |
| MinorVault.sol | One locked account per minor. Released once, in full, at 18 to the minor's own wallet (or to the guardian on death), always with 7 days' public notice; CUPCONL or the treasury can stop a release. |

USDT is paid through the Asset Hub ERC-20 precompile, so it arrives as a normal USDT balance in the resident's Polkadot wallet.

### Wallet

Our own self-custodial wallet in the Decidimos app. Keys stay on the phone. Before payouts scale up, it is tested by Heitaria with the communities and by an independent security firm. Fees for the residents' own payments are paid in USDT, so no resident ever needs DOT. Heitaria activates each new wallet with 0.05 DOT and pays the fees of the monthly payouts.

### Secret ballot

Votes use Chaum RSA-2048 blind signatures (full-domain hash, MGF1/SHA-256): one token per voter per voting cycle, issued and spent in two separate phases, with a nullifier against double voting. The protocol is designed so that the server cannot link a ballot to a voter. Independent cryptographic review by Hacken in April 2027.

### Minors' savings

For every child, the parents receive 70% of the child's monthly share until the child turns 18. The other 30% stays locked in MinorVault. 15% of the locked amount is lent as USDT in Hydration's money market; the interest is added to the child's account. This starts only after the external audit and with CUPCONL's approval.

## Milestones

Work runs from January to June 2027. 190,000 USDT in eight monthly tranches, one a month, February to September 2027:

| # | Milestone | USDT | Claimable from |
|---|---|---|---|
| 1 | Payout contracts, payout engine and panel | 40,000 | February 2027 |
| 2 | Enrolment desk, assemblies, loading the list | 20,000 | March 2027 |
| 3 | Wallet and recovery on mainnet | 20,000 | April 2027 |
| 4 | Records on Polkadot, public check page | 15,000 | May 2027 |
| 5 | Audit and load test | 45,000 | June 2027 |
| 6 | Facilitators running the cycle | 15,000 | July 2027 |
| 7 | Shop mode, cash-out | 15,000 | August 2027 |
| 8 | Test month, children's vault, open source, report | 20,000 | September 2027 |
| | **Total** | **190,000** | |

Each milestone is paid only from its date, after the delivery is posted in the forum thread. Progress is tracked in the issues of this repository.

## Documents

- [Plan for spending the 190,000 USDT](docs/Spending_plan_190000_USDT.md)
- [Step-by-step plan, January – September 2027](docs/Step_by_step_plan.md)
- [Operations manual (English)](docs/Decidimos-Operations-manual.pdf) · [Manual de operación (español)](docs/Decidimos-Manual-de-operacion.pdf) · [Guía para la comunidad](docs/Decidimos-Guia-para-la-comunidad.pdf)
- Memoranda, signature pages: [Río Grande](docs/Memorandum_Rio_Grande_firmas.jpg) · [CUPCONL](docs/Memorandum_CUPCONL_firmas.jpg)

## Code

The contracts, the payout engine and the wallet code are published here after the external audit (milestone 5) and at the latest with milestone 8, together with the audit report.

## Team

Heitaria Swiss AG, Rigistrasse 1, 6374 Buochs, Switzerland, trading since 2013.
Paul Schmidt (CEO).
schmidt@heitaria.ch · +41 78 339 59 77

## Licence

Apache-2.0 for the code. The documents in docs/ are not covered by this licence: © Heitaria Swiss AG, all rights reserved.
