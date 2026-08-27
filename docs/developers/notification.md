From 2.5 version

# Notification en absence de session, gestion de l'asynchronisme des messages

## Notifications persistantes en base de données

Au lieu d'utiliser les flash messages (qui nécessitent une session), nous allons **stocker les notifications dans la base de données**.

---

## Étapes d'implémentation

### 1. Créer l'entité UserNotification

Créez le fichier `src/Entity/UserNotification.php` avec le contenu fourni.

### 2. Créer la migration

```bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
```

Cela créera la table `user_notifications` dans votre base de données.

### 3. Créer le service NotificationService

Créez `src/Service/NotificationService.php` avec le contenu fourni.

Ce service gère :
- ✅ Création de notifications (fonctionne sans session)
- ✅ Récupération des notifications non lues
- ✅ Marquage comme lu
- ✅ Nettoyage des anciennes notifications

### 4. Créer l'extension Twig

Créez `src/Twig/NotificationExtension.php` avec le contenu fourni.

Cette extension permet d'accéder aux notifications dans les templates :
```twig
{% set notifications = get_unread_notifications() %}
{% set count = get_notification_count() %}
```

### 5. Créer le contrôleur NotificationController

Créez `src/Controller/NotificationController.php` avec le contenu fourni.

Ce contrôleur gère les endpoints AJAX pour marquer les notifications comme lues.

### 6. Mettre à jour InstanceStateMessageHandler

Remplacez votre `src/MessageHandler/InstanceStateMessageHandler.php` par la version fournie dans `InstanceStateMessageHandler_fixed.php`.

**Changements principaux :**
```php
// AVANT (ne fonctionne pas)
private RequestStack $requestStack;
$this->addFlashMessage('danger', 'Erreur');

// APRÈS (fonctionne !)
private NotificationService $notificationService;
$this->notificationService->error($userId, 'Erreur', $uuid);
```

### 7. Mettre à jour le template dashboard

Mettez à jour `templates/dashboard.base.html.twig` pour afficher les notifications persistantes :

```twig
{# Display Persistent Notifications from Database #}
{% set unread_notifications = get_unread_notifications() %}
{% if unread_notifications is not empty %}
  <div class="persistent-notifications" data-notifications="{{ unread_notifications|json_encode|e('html_attr') }}">
    {# Notifications will be displayed via JavaScript using react-toastify #}
  </div>
{% endif %}
```

### 8. Mettre à jour app.js

Remplacez `assets/js/app.js` par le contenu de `app_with_persistent_notifications.js`.

**Changements :**
- Affiche les flash messages traditionnels (pour les autres pages)
- Affiche les notifications persistantes de la base de données
- Marque automatiquement les notifications comme lues via AJAX

### 9. Configurer les services (optionnel)

Si l'autowiring ne fonctionne pas automatiquement, ajoutez dans `config/services.yaml` :

```yaml
services:
    App\Service\NotificationService:
        autowire: true
        public: false

    App\Twig\NotificationExtension:
        tags: ['twig.extension']
```

---

## Comment ça fonctionne

### Flux de notification

```
1. InstanceStateMessageHandler (worker asynchrone)
   ↓
2. NotificationService crée une notification en DB
   ↓
3. Utilisateur charge une page
   ↓
4. Template récupère les notifications via get_unread_notifications()
   ↓
5. JavaScript affiche les notifications avec react-toastify
   ↓
6. AJAX marque la notification comme lue
```

### Avantages de cette solution

✅ **Fonctionne sans session** (parfait pour les workers asynchrones)
✅ **Notifications persistantes** (pas perdues si l'utilisateur ne voit pas la page immédiatement)
✅ **Historique** (possibilité de consulter les anciennes notifications)
✅ **Multi-utilisateurs** (chaque utilisateur voit ses propres notifications)
✅ **Compatible avec votre code existant** (react-toastify est déjà installé)

---

## Utilisation

### Dans un MessageHandler

```php
use App\Service\NotificationService;

class MyMessageHandler
{
    private NotificationService $notificationService;

    public function __construct(NotificationService $notificationService)
    {
        $this->notificationService = $notificationService;
    }

    public function __invoke(MyMessage $message)
    {
        // Récupérer l'ID utilisateur (selon votre logique)
        $userId = $message->getUserId();
        
        // Ajouter une notification
        $this->notificationService->error(
            $userId,
            'Une erreur est survenue',
            $relatedUuid // optionnel
        );
        
        // Ou
        $this->notificationService->success($userId, 'Opération réussie !');
        $this->notificationService->warning($userId, 'Attention...');
        $this->notificationService->info($userId, 'Information');
    }
}
```

### Dans un contrôleur (avec session disponible)

```php
use App\Service\NotificationService;

class IsoController extends AbstractController
{
    public function __construct(
        private NotificationService $notificationService
    ) {}

    #[Route('/iso/delete/{id}')]
    public function delete(int $id): Response
    {
        $user = $this->getUser();
        
        try {
            // Logique de suppression
            // ...
            
            $this->notificationService->success(
                (string) $user->getId(),
                'ISO supprimé avec succès'
            );
        } catch (\Exception $e) {
            $this->notificationService->error(
                (string) $user->getId(),
                'Erreur lors de la suppression : ' . $e->getMessage()
            );
        }
        
        return $this->redirectToRoute('app_iso_index');
    }
}
```

---

## Nettoyage automatique

Pour nettoyer automatiquement les anciennes notifications lues, créez une commande :

```php
// src/Command/CleanNotificationsCommand.php

namespace App\Command;

use App\Service\NotificationService;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class CleanNotificationsCommand extends Command
{
    protected static $defaultName = 'app:notifications:clean';
    
    public function __construct(
        private NotificationService $notificationService
    ) {
        parent::__construct();
    }

    protected function execute(InputInterface $input, OutputInterface $output): int
    {
        $this->notificationService->deleteOldNotifications(30);
        $output->writeln('Old notifications cleaned successfully');
        return Command::SUCCESS;
    }
}
```

Puis ajoutez dans crontab :
```bash
# Nettoyer les notifications tous les jours à 2h du matin
0 2 * * * cd /path/to/project && php bin/console app:notifications:clean
```

---

## Test de la solution

### 1. Tester depuis un MessageHandler

```php
// Ajoutez temporairement dans votre handler
$this->notificationService->error(
    '1', // ID utilisateur (remplacez par un vrai ID)
    'Test de notification',
    'test-uuid'
);
```

### 2. Déclencher le handler

```bash
# Si vous utilisez Symfony Messenger
php bin/console messenger:consume async -vv
```

### 3. Vérifier la base de données

```sql
SELECT * FROM user_notifications ORDER BY created_at DESC LIMIT 5;
```

### 4. Charger une page

Connectez-vous avec l'utilisateur et chargez n'importe quelle page.
Vous devriez voir la notification s'afficher avec react-toastify !

---

## Dépannage

### Les notifications ne s'affichent pas

1. **Vérifiez la base de données**
   ```sql
   SELECT * FROM user_notifications WHERE is_read = 0;
   ```

2. **Vérifiez la console JavaScript (F12)**
   - Y a-t-il des erreurs ?
   - La variable `notifications` est-elle présente dans le HTML ?

3. **Vérifiez que l'extension Twig est chargée**
   ```bash
   php bin/console debug:twig
   # Doit afficher get_unread_notifications
   ```

### Erreur "User ID null"

C'est normal pour certaines instances qui n'ont pas d'utilisateur associé.
Les notifications avec `userId = null` ne seront affichées à personne.

Adaptez la méthode `getUserIdFromInstance()` selon votre modèle de données.

### Les notifications ne sont pas marquées comme lues

1. **Vérifiez la route** dans `NotificationController`
2. **Vérifiez les logs réseau** dans la console (F12 > Network)
3. **Vérifiez que jQuery est chargé** avant le script

---

## Résumé

**Problème :** FlashMessages ne fonctionnent pas dans les MessageHandlers asynchrones

**Solution :** Notifications persistantes en base de données

**Avantages :**
- ✅ Fonctionne avec ou sans session
- ✅ Notifications persistantes
- ✅ Compatible avec votre système existant (react-toastify)
- ✅ Facile à étendre

**Fichiers à créer/modifier :**
1. `src/Entity/UserNotification.php` (nouveau)
2. `src/Service/NotificationService.php` (nouveau)
3. `src/Twig/NotificationExtension.php` (nouveau)
4. `src/Controller/NotificationController.php` (nouveau)
5. `src/MessageHandler/InstanceStateMessageHandler.php` (modifier)
6. `templates/dashboard.base.html.twig` (modifier)
7. `assets/js/app.js` (modifier)

C'est parti ! 🚀