# Windows Enterprise Infrastructure Lab

Laboratoire d'infrastructure Windows virtualisé : domaine Active Directory, DNS, DHCP, partages de fichiers par service et stratégies de groupe. Projet réalisé dans le cadre de mon BTS SIO option SISR (IFC Avignon).

**Dossier technique complet (35 pages, français) :** [`docs/Khaltech_Solutions_Documentation.pdf`](docs/Khaltech_Solutions_Documentation.pdf)

## Objectif

Construire de bout en bout l'infrastructure Windows d'une petite entreprise fictive (Khaltech Solutions) et prouver chaque résultat par un test et une capture d'écran.

## Environnement

| Élément | Valeur |
|---|---|
| Hyperviseur | Oracle VirtualBox, réseau interne `KHALTECH-LAB` |
| Contrôleur de domaine | `KTS-DC01` : Windows Server 2022 Standard (évaluation), `192.168.10.10` |
| Poste client | `KTS-CLIENT01` : Windows 11, membre du domaine |
| Domaine | `khaltech.local` (nouvelle forêt) |
| Réseau | `192.168.10.0/24`, pool DHCP `.100` à `.200`, options 006 (DNS) et 015 (domaine) |

![Machines virtuelles](screenshots/Machines-within-the-Infra.png)

## Ce qui a été réalisé

- **Active Directory / DNS** : promotion de KTS-DC01 en contrôleur de domaine (catalogue global), service DNS vérifié.
- **DHCP** : étendue `KHALTECH-LAN`, options DNS et nom de domaine.
- **Structure de l'annuaire** : OU `KHALTECH` avec `COMPUTERS`, `GROUPS`, `USERS` (HR, IT, MANAGEMENT) ; groupes `GG_HR_Staff` et `GG_IT_Support`.
- **Partages SMB** `HR` et `IT` : permissions de partage (Change) et NTFS (Modify) séparées, accordées aux groupes de service, héritage désactivé.
- **GPO ordinateur** `GPO-Workstation-Security-Baseline` liée à l'OU `COMPUTERS` (pare-feu activé pour le profil de domaine).

![Structure AD](screenshots/01-windows-enterprise.png)

## Tests de validation

| Test | Résultat |
|---|---|
| Accès au partage HR par Sarah (groupe `GG_HR_Staff`) | Accès autorisé, modification d'un fichier |
| Accès au partage HR par Alex (service IT) | Accès refusé |
| `gpresult` sur le poste client | `GPO-Workstation-Security-Baseline` et `Default Domain Policy` appliquées |

![Permissions HR](screenshots/01-07-HR-SMB-and-NTFS-Permissions.png)
![Alex refusé](screenshots/01-09-Alex-Denied-HR-Share.png)
![gpresult](screenshots/06-computer-gpo-gpresult.png)

## Incident résolu : « Accès refusé » à la création du partage HR

- **Symptôme** : création du partage impossible, dans l'assistant comme avec `New-SmbShare` (erreur système 5), alors que j'étais administrateur du domaine.
- **Hypothèses écartées** : droits de l'opérateur, service Serveur arrêté, SMB désactivé.
- **Cause racine** : l'ACL NTFS de `C:\Shares\HR` ne contenait que `GG_HR_Staff`. Le compte SYSTEM, sous lequel tourne le service Serveur, n'avait plus accès au dossier.
- **Correction** : ajout de SYSTEM et Administrateurs en contrôle total avec `icacls` (par SID, indépendant de la langue), héritage laissé désactivé, puis `New-SmbShare`.
- **Leçon** : toujours garder SYSTEM et Administrateurs dans l'ACL d'un dossier partagé, régler le NTFS avant de créer le partage.

```powershell
icacls "C:\Shares\hr" /grant "*S-1-5-18:(OI)(CI)F" "*S-1-5-32-544:(OI)(CI)F"
New-SmbShare -Name "HR" -Path "C:\Shares\HR" -ChangeAccess "KHALTECH\GG_HR_Staff" -FullAccess "BUILTIN\Administrateurs"
```

## Limites et suite

- Un seul contrôleur de domaine, réseau isolé sans accès Internet, Windows Server en version d'évaluation.
- En cours de documentation : vérification des baux DHCP, tests d'accès au partage IT, GPO utilisateur du service IT.
- Prochains projets : laboratoire réseau, helpdesk GLPI, laboratoire Linux.

## Contact

Yassine Khalfallah, étudiant BTS SIO SISR à Avignon, en recherche d'alternance en support informatique ou systèmes et réseaux.
LinkedIn : [linkedin.com/in/yassine-khalfallah](https://www.linkedin.com/in/yassine-khalfallah/)
