# Securisation Active Directory : Dangers et Remediations

Ce document repertorie les principales failles de configuration par defaut d'un environnement Active Directory, explique comment les attaquants les exploitent, et detaille les actions correctives a appliquer (generalement via GPO).

## Sommaire

1. [Desactivation LLMNR et NetBIOS (NBT-NS)](#1-desactivation-llmnr-et-netbios-nbt-ns)
2. [Signature SMB (SMB Signing)](#2-signature-smb-smb-signing)
3. [Securite LDAP (LDAP Signing et Channel Binding)](#3-securite-ldap-ldap-signing-et-channel-binding)
4. [Mots de passe administrateurs locaux (LAPS)](#4-mots-de-passe-administrateurs-locaux-laps)

## 1. Desactivation LLMNR et NetBIOS (NBT-NS)

Protocoles de resolution de noms obsoletes utilises par Windows lorsque le serveur DNS ne repond pas.

<div style="background: linear-gradient(135deg, #fff3f3 0%, #ffe5e5 100%); border: 1px solid #f6b6b6; border-left: 8px solid #b91c1c; color: #3f0f0f; padding: 16px; border-radius: 10px; margin: 14px 0; box-shadow: 0 4px 14px rgba(127, 29, 29, 0.12);">
<h3 style="margin: 0 0 10px 0; color: #7f1d1d;">Danger : LLMNR/NBT-NS Poisoning</h3>
<p>Lorsqu'une machine tente de joindre un hote inexistant ou mal orthographie (exemple : <code style="background: #ffd9d9; color: #7f1d1d; padding: 1px 6px; border-radius: 4px;">\\SERVEUR-PARTAG</code>), la requete DNS echoue.</p>
<p>Windows envoie alors une requete en broadcast sur le reseau local via LLMNR et NetBIOS :</p>
<blockquote style="margin: 10px 0; padding: 8px 12px; border-left: 4px solid #ef4444; background: #ffe9e9; color: #4a1212; border-radius: 6px;">
<p>"Qui est SERVEUR-PARTAG ?"</p>
</blockquote>
<p>Un attaquant sur le meme reseau (via un outil comme Responder) peut intercepter cette requete et repondre :</p>
<blockquote style="margin: 10px 0; padding: 8px 12px; border-left: 4px solid #ef4444; background: #ffe9e9; color: #4a1212; border-radius: 6px;">
<p>"C'est moi, envoie tes identifiants pour te connecter"</p>
</blockquote>
<p>La machine victime envoie automatiquement le condensat du mot de passe de l'utilisateur (hash NTLMv2). L'attaquant peut ensuite tenter de craquer ce hash hors ligne ou l'utiliser pour une attaque de relais.</p>
</div>

<div style="background: linear-gradient(135deg, #f3fdf5 0%, #e3f8e8 100%); border: 1px solid #b7e3c3; border-left: 8px solid #15803d; color: #0f3b20; padding: 16px; border-radius: 10px; margin: 14px 0; box-shadow: 0 4px 14px rgba(21, 128, 61, 0.12);">
<h3 style="margin: 0 0 10px 0; color: #166534;">Remediation</h3>
<p>Si votre infrastructure DNS est fonctionnelle, ces protocoles sont inutiles et dangereux.</p>
<ol>
<li><strong>Desactiver LLMNR (via GPO)</strong>
<ul>
<li><strong>Chemin</strong> : Configuration ordinateur &gt; Strategies &gt; Modeles d'administration &gt; Reseau &gt; Client DNS</li>
<li><strong>Parametre</strong> : Desactiver la resolution de noms multidiffusion -&gt; Active</li>
</ul>
</li>
<li><strong>Desactiver NetBIOS sur TCP/IP</strong>
<ul>
<li><strong>Via DHCP</strong> : configurer l'option d'etendue <code style="background: #d8f5df; color: #166534; padding: 1px 6px; border-radius: 4px;">001</code> (Disable Netbios over TCP/IP)</li>
<li><strong>Via script (GPO ou RMM)</strong> : NetBIOS est lie a la carte reseau, il faut donc executer un script PowerShell pour le desactiver sur toutes les interfaces actives</li>
</ul>
</li>
</ol>
</div>

## 2. Signature SMB (SMB Signing)

Mecanisme garantissant l'integrite des communications Server Message Block (utilise pour les partages de fichiers et les IPC).

<div style="background: linear-gradient(135deg, #fff3f3 0%, #ffe5e5 100%); border: 1px solid #f6b6b6; border-left: 8px solid #b91c1c; color: #3f0f0f; padding: 16px; border-radius: 10px; margin: 14px 0; box-shadow: 0 4px 14px rgba(127, 29, 29, 0.12);">
<h3 style="margin: 0 0 10px 0; color: #7f1d1d;">Danger : SMB Relay</h3>
<p>Par defaut, sur les postes de travail et serveurs membres, la signature SMB est activee mais non requise (elle n'est requise que sur les controleurs de domaine).</p>
<p>Si un attaquant intercepte une tentative de connexion NTLM (souvent couplee a l'attaque LLMNR ci-dessus), il peut relayer cette session authentifiee en temps reel vers une autre machine cible, sans craquer le mot de passe. Si l'utilisateur intercepte possede des droits d'administration sur la machine cible, l'attaquant peut en prendre le controle total.</p>
</div>

<div style="background: linear-gradient(135deg, #f3fdf5 0%, #e3f8e8 100%); border: 1px solid #b7e3c3; border-left: 8px solid #15803d; color: #0f3b20; padding: 16px; border-radius: 10px; margin: 14px 0; box-shadow: 0 4px 14px rgba(21, 128, 61, 0.12);">
<h3 style="margin: 0 0 10px 0; color: #166534;">Remediation</h3>
<p>Rendre la signature SMB obligatoire empeche le relais, car l'attaquant ne peut pas signer numeriquement les paquets modifies sans posseder la cle de session.</p>
<p><strong>Configuration via GPO</strong></p>
<ul>
<li><strong>Chemin</strong> : Configuration ordinateur &gt; Strategies &gt; Parametres Windows &gt; Parametres de securite &gt; Strategies locales &gt; Options de securite</li>
<li><strong>Client reseau Microsoft : Signer numeriquement les communications (toujours)</strong> -&gt; Active</li>
<li><strong>Serveur reseau Microsoft : Signer numeriquement les communications (toujours)</strong> -&gt; Active</li>
</ul>
<blockquote style="margin: 10px 0; padding: 8px 12px; border-left: 4px solid #22c55e; background: #e8f9ed; color: #14532d; border-radius: 6px;">
<p>Note : peut avoir un leger impact sur les performances lors de transferts de tres gros fichiers. A tester avant deploiement global.</p>
</blockquote>
</div>

## 3. Securite LDAP (LDAP Signing et Channel Binding)

Protection des requetes de lecture et de modification vers l'annuaire Active Directory.

<div style="background: linear-gradient(135deg, #fff3f3 0%, #ffe5e5 100%); border: 1px solid #f6b6b6; border-left: 8px solid #b91c1c; color: #3f0f0f; padding: 16px; border-radius: 10px; margin: 14px 0; box-shadow: 0 4px 14px rgba(127, 29, 29, 0.12);">
<h3 style="margin: 0 0 10px 0; color: #7f1d1d;">Danger : LDAP Relay et trafic en clair</h3>
<p>Par defaut, un AD accepte les requetes LDAP simples (port 389), non chiffrees et non signees.</p>
<ul>
<li><strong>Ecoute reseau</strong> : si des applications tierces s'authentifient en LDAP simple, les identifiants transitent en clair</li>
<li><strong>LDAP relaying</strong> : un attaquant peut relayer une authentification NTLM interceptee (HTTP ou SMB) vers le controleur de domaine via LDAP et demander des modifications d'attributs critiques (exemple : ajout machine au domaine, modification de delegations Kerberos)</li>
</ul>
</div>

<div style="background: linear-gradient(135deg, #f3fdf5 0%, #e3f8e8 100%); border: 1px solid #b7e3c3; border-left: 8px solid #15803d; color: #0f3b20; padding: 16px; border-radius: 10px; margin: 14px 0; box-shadow: 0 4px 14px rgba(21, 128, 61, 0.12);">
<h3 style="margin: 0 0 10px 0; color: #166534;">Remediation</h3>
<p>Il faut imposer la signature des paquets LDAP et securiser le canal TLS (LDAPS).</p>
<p><strong>Configuration sur les controleurs de domaine (via la GPO "Default Domain Controllers Policy")</strong></p>
<ul>
<li><strong>Chemin</strong> : Configuration ordinateur &gt; Strategies &gt; Parametres Windows &gt; Parametres de securite &gt; Strategies locales &gt; Options de securite</li>
<li><strong>Controleur de domaine : Exigences de signature du serveur LDAP</strong> -&gt; Requis</li>
<li><strong>Serveur reseau Microsoft : Exigences de liaison de canal (Channel Binding) LDAP</strong> -&gt; Toujours</li>
</ul>
<blockquote style="margin: 10px 0; padding: 8px 12px; border-left: 4px solid #22c55e; background: #e8f9ed; color: #14532d; border-radius: 6px;">
<p>Note : attention aux anciennes applications, copieurs ou firewalls qui ne supportent pas LDAPS. Un audit des journaux d'evenements LDAP (Event ID 2889) est recommande avant d'activer le blocage.</p>
</blockquote>
</div>

## 4. Mots de passe administrateurs locaux (LAPS)

Gestion des comptes administrateurs locaux integres aux machines Windows.

<div style="background: linear-gradient(135deg, #fff3f3 0%, #ffe5e5 100%); border: 1px solid #f6b6b6; border-left: 8px solid #b91c1c; color: #3f0f0f; padding: 16px; border-radius: 10px; margin: 14px 0; box-shadow: 0 4px 14px rgba(127, 29, 29, 0.12);">
<h3 style="margin: 0 0 10px 0; color: #7f1d1d;">Danger : mouvement lateral et Pass-The-Hash</h3>
<p>Dans de nombreuses infrastructures, les postes de travail ou serveurs sont deployes via une image systeme commune. Le mot de passe du compte administrateur local (exemple : <code style="background: #ffd9d9; color: #7f1d1d; padding: 1px 6px; border-radius: 4px;">.\Administrateur</code>) est alors identique sur toutes les machines.</p>
<p>Si un attaquant compromet un seul poste (phishing, malware, etc.), il peut extraire le hash du mot de passe administrateur local (exemple : via Mimikatz). Comme ce hash est valable sur les autres machines, il peut l'utiliser en Pass-The-Hash pour rebondir lateralement sur le parc.</p>
</div>

<div style="background: linear-gradient(135deg, #f3fdf5 0%, #e3f8e8 100%); border: 1px solid #b7e3c3; border-left: 8px solid #15803d; color: #0f3b20; padding: 16px; border-radius: 10px; margin: 14px 0; box-shadow: 0 4px 14px rgba(21, 128, 61, 0.12);">
<h3 style="margin: 0 0 10px 0; color: #166534;">Remediation</h3>
<p>Il ne faut jamais reutiliser le meme mot de passe local sur plusieurs machines, ni le deployer via GPO (GPP).</p>
<p><strong>Deployer LAPS (Local Administrator Password Solution)</strong></p>
<ul>
<li>Microsoft LAPS (ou Windows LAPS integre aux OS recents) genere un mot de passe aleatoire, unique et complexe pour chaque machine</li>
<li>Ce mot de passe est renouvele automatiquement (exemple : tous les 30 jours)</li>
<li>Il est stocke de maniere chiffree dans un attribut de l'objet Ordinateur dans Active Directory</li>
<li>Seuls les administrateurs de domaine (ou groupes explicitement autorises) peuvent lire cet attribut en cas de besoin</li>
</ul>
</div>
