🛠️  Apex Backend (Simulation locale)
Ce projet est une tentative de recréer un backend simulé pour le jeu Apex Legends, dans le but de déboguer, comprendre et simuler les appels réseau effectués par le launcher et le jeu, sans se connecter aux serveurs réels d’EA/Respawn.

📌 Objectifs
Fournir un backend local simulé sur http://127.0.0.1:3000

Permettre à un launcher custom d’effectuer un login basique avec génération de token factice (MOCK_AUTH_CODE_12345)

Rediriger tous les domaines EA/Respawn vers le serveur local à l’aide de Fiddler

✅ Fonctionnalités déjà en place
✅ Serveur Express.js local sur le port 3000

✅ Route /connect/auth pour simuler un login OAuth très basique

✅ Réponse JSON simulée au lieu de vraie redirection (utile pour le debug)

✅ Fichier game_launcher.html pour simuler une interface de connexion

✅ Script Fiddler complet pour capturer et rediriger les appels réseau du jeu

⚠️ Limites actuelles
❌ Le jeu bloque sur le code Easy Anti-Cheat (EAC). Aucune solution n'est encore en place pour le bypass.

❌ Certaines routes (/sdk/v1/default, etc.) doivent encore être simulées pour correspondre aux attentes du launcher.

⚠️ Ce projet est entièrement hors-ligne et ne permet pas de jouer à Apex Legends. Il ne s'agit pas d'un crack, mais d'une simulation pour étude.

🧪 Exemple de réponse simulée (/connect/auth)

{
  "message": "Authorization successful - use the code to request token",
  "redirect_to": "qrc:///html/login_successful.html?code=MOCK_AUTH_CODE_12345",
  "authorization_code": "MOCK_AUTH_CODE_12345"
}
🧩 Redirection via Fiddler (script)
Voici le script Fiddler complet que tu peux coller dans Rules > Customize Rules... > OnBeforeRequest pour rediriger tous les domaines EA/Respawn vers localhost:3000 :


import System;
import System.IO;
import System.Threading;
import System.Web;
import System.Windows.Forms;
import Fiddler;

class Handlers
{
    static var apexDomains : String[] = [
        "gateway.ea.com",
        "api.mediacdn.ea.com",
        "playerdata.respawn.com",
        "live.respawn.com",
        "apex-api.respawn.com",
        "api.epicgames.dev",
        "privacy.xboxlive.com",
        "epicgames.dev",
        "bam.nr-data.net",
        "trm.tnt-ea.com:8095",
        "drivefrontend-pa.clients6.google.com",
        "pc.ea.com",
        "accounts.ea.com",
        "confluence.ea.com",
        "app-images.ea.com",
        "content-origin.ea.com",
        "origin-a.akamaihd.net",
        "origin.ea.com",
        "eaassets-a.akamaihd.net",
        "eaassets-origin.akamaized.net",
        "stats.ea.com",
        "ea.gssprt.com",
        "ea.gssprt.net",
        "ea-prod.apexstats.net",
        "apex.tracker.gg",
        "api1.respawn.com",
        "gssprt.net",
        "live1.respawn.com",
        "apexwebstats.respawn.com",
        "social.ea.com",
        "accounts.epicgames.com",
        "account-public-service-prod03.ol.epicgames.com"
    ];

    static function OnBeforeRequest(oSession: Session) {
        for (var i:int = 0; i < apexDomains.Length; i++) {
            var domain = apexDomains[i];

            var hostToCheck = oSession.hostname.ToLower();
            if (domain.Contains(":")) {
                hostToCheck = hostToCheck + ":" + oSession.port.ToString();
            }

            if (hostToCheck.Contains(domain.ToLower())) {
                if (oSession.HTTPMethodIs("CONNECT")) {
                    oSession["x-replywithtunnel"] = "FakeTunnel";
                    return;
                }

                var newUrl = "http://127.0.0.1:3000" + oSession.PathAndQuery;
                oSession.fullUrl = newUrl;
                oSession.oRequest["Host"] = "127.0.0.1:3000";

                FiddlerApplication.Log.LogString("[APEX REDIRECT] " + domain + " => " + newUrl);
                break;
            }
        }
    }
}
🚧 À faire / Roadmap
 Reproduire les endpoints manquants (/sdk/v1/default, /account, /lobby, etc.)

 Simuler un token JWT complet

 Gérer les headers User-Agent, Authorization, etc.

 Étudier le comportement du launcher pour mimer l’échange réel

 Étudier Easy Anti-Cheat pour comprendre la vérification bloquante

🧠 But du projet
Ce projet n’a pas pour objectif de contourner la sécurité du jeu, mais bien d’étudier son architecture réseau, dans un but d’apprentissage, d’expérimentation et de développement de launchers custom pour l’environnement local/sandbox.
