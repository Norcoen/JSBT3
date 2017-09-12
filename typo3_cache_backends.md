# TYPO3 mit APCu im Einsatz
Quelle: https://rosenauer.blog/typo3-mit-apcu-im-einsatz/ - 2017--02-22

TYPO3 unterstützt schon von Haus aus APCu.

Die Aktivierung erfolgt in der **LocalConfiguration.php** unter **/typo3conf/** oder direkt im **Install-Tool** unter **„Configuration Presets“** (ab TYPO3 6.2).

*Diese Aktivierung bringt jedoch nicht sonderlich viel Performance. Mit diesem Code beschleunigen Sie Ihr TYPO3 300 prozentig.*

Fügen Sie den Code in Ihre **AdditionalConfiguration.php** unter **/typo3conf/**. *Wenn die Datei nicht existiert, legen Sie diese an:*

```php
<?php
// File: /html/typo3/typo3conf/AdditionalConfiguration.php
 
function mw_setCacheBackend($backendClassName, $cacheName) {
        $GLOBALS['TYPO3_CONF_VARS']['SYS']['caching']['cacheConfigurations'][$cacheName]['backend'] = $backendClassName;
        $GLOBALS['TYPO3_CONF_VARS']['SYS']['caching']['cacheConfigurations'][$cacheName]['options'] = array();
}
$mw_apcExtensionLoaded = extension_loaded('apc');
$mw_apcuExtensionLoaded = extension_loaded('apcu');
$mw_apcAvailable = $mw_apcExtensionLoaded || $mw_apcuExtensionLoaded;
$mw_apcEnabled = ini_get('apc.enabled') == TRUE;
$mw_backendClassName = 'TYPO3\\CMS\\Core\\Cache\\Backend\\FileBackend';
 
if (PHP_SAPI !== 'cli' && TYPO3\CMS\Core\Utility\GeneralUtility::getApplicationContext() !== 'Development' && $mw_apcAvailable && $mw_apcEnabled)
{
        $mw_backendClassName = $mw_apcExtensionLoaded ? 'TYPO3\\CMS\\Core\\Cache\\Backend\\ApcBackend' : 'TYPO3\\CMS\\Core\\Cache\\Backend\\ApcuBackend';
 
        mw_setCacheBackend($mw_backendClassName, 'cache_hash');
        mw_setCacheBackend($mw_backendClassName, 'cache_hash_tags');
        mw_setCacheBackend($mw_backendClassName, 'cache_imagesizes');
        mw_setCacheBackend($mw_backendClassName, 'cache_imagesizes_tags');
        mw_setCacheBackend($mw_backendClassName, 'cache_pages');
        mw_setCacheBackend($mw_backendClassName, 'cache_pages_tags');
        mw_setCacheBackend($mw_backendClassName, 'cache_pagesection');
        mw_setCacheBackend($mw_backendClassName, 'cache_pagesection_tags');
        mw_setCacheBackend($mw_backendClassName, 'cache_rootline');
        mw_setCacheBackend($mw_backendClassName, 'cache_rootline_tags');
        mw_setCacheBackend($mw_backendClassName, 'extbase_datamapfactory_datamap');
        mw_setCacheBackend($mw_backendClassName, 'extbase_datamapfactory_datamap_tags');
        mw_setCacheBackend($mw_backendClassName, 'extbase_object');
        mw_setCacheBackend($mw_backendClassName, 'extbase_object_tags');
        mw_setCacheBackend($mw_backendClassName, 'extbase_reflection');
        mw_setCacheBackend($mw_backendClassName, 'extbase_reflection_tags');
        mw_setCacheBackend($mw_backendClassName, 'extbase_typo3dbbackend_queries');
        mw_setCacheBackend($mw_backendClassName, 'extbase_typo3dbbackend_queries_tags');
        mw_setCacheBackend($mw_backendClassName, 'extbase_typo3dbbackend_tablecolumns');
        mw_setCacheBackend($mw_backendClassName, 'extbase_typo3dbbackend_tablecolumns_tags');
}
```
Wenn der TYPO3-Scheduler oder andere Cronjobs in TYPO3 nicht funktionieren, kann es sein, dass TYPO3 versucht diese über APC aufzurufen.
**Hierfür fügen wir weiteren Code der AdditionalConfiguration.php hinzu:**
```php
<?php
if (PHP_SAPI === 'cli') {
// cache config for CLI here
$GLOBALS['TYPO3_CONF_VARS']['SYS']['caching']['cacheConfigurations']['extbase_object']['backend'] = 'TYPO3\\CMS\\Core\\Cache\\Backend\\Typo3DatabaseBackend';
}
?>
```

---

# APCu mit TYPO3 verwenden
Quelle: https://www.mittwald.de/faq/frage/apcu-mit-typo3-verwenden - unbekanntes Einstellungsdatum. gesichtet 2017-09-12

In Ihrer TYPO3-Installation ist die Verwendung des APCu durch Performance Plus noch nicht optimal eingerichtet. Um den maximalen Effekt zu erzielen, können Sie TYPO3 anweisen, das Caching durch APCu vorzunehmen anstatt durch die Datenbank.

Erstellen Sie dafür bitte im Verzeichnis typo3conf Ihrer TYPO3-Installation eine AdditionalConfiguration.php und fügen Sie dort die folgenden Zeilen ein:
```php
<?php

if (!function_exists('mw_setCacheBackend')) {
    function mw_setCacheBackend($backendClassName, $cacheName)
    {
        $GLOBALS['TYPO3_CONF_VARS']['SYS']['caching']['cacheConfigurations'][$cacheName]['backend'] = $backendClassName;
        $GLOBALS['TYPO3_CONF_VARS']['SYS']['caching']['cacheConfigurations'][$cacheName]['options'] = array();
    }
}

$mw_apcExtensionLoaded = extension_loaded('apc');
$mw_apcuExtensionLoaded = extension_loaded('apcu');
$mw_apcAvailable = $mw_apcExtensionLoaded || $mw_apcuExtensionLoaded;
$mw_apcEnabled = ini_get('apc.enabled') == TRUE;

if (TYPO3\CMS\Core\Utility\GeneralUtility::getApplicationContext() !== 'Development' && $mw_apcAvailable && $mw_apcEnabled) {
    $mw_backendClassName = $mw_apcExtensionLoaded ? 'TYPO3\\CMS\\Core\\Cache\\Backend\\ApcBackend'
        : 'TYPO3\\CMS\\Core\\Cache\\Backend\\ApcuBackend';
} else {
    $mw_backendClassName = 'TYPO3\\CMS\\Core\\Cache\\Backend\\FileBackend';
}

mw_setCacheBackend($mw_backendClassName, 'cache_hash');
mw_setCacheBackend($mw_backendClassName, 'cache_imagesizes');
mw_setCacheBackend($mw_backendClassName, 'cache_pages');
mw_setCacheBackend($mw_backendClassName, 'cache_pagesection');
mw_setCacheBackend($mw_backendClassName, 'cache_rootline');
mw_setCacheBackend($mw_backendClassName, 'extbase_datamapfactory_datamap');
mw_setCacheBackend($mw_backendClassName, 'extbase_object');
mw_setCacheBackend($mw_backendClassName, 'extbase_reflection');
mw_setCacheBackend($mw_backendClassName, 'extbase_typo3dbbackend_queries');
mw_setCacheBackend($mw_backendClassName, 'extbase_typo3dbbackend_tablecolumns');

if (PHP_SAPI === 'cli') {
    $GLOBALS['TYPO3_CONF_VARS']['SYS']['caching']['cacheConfigurations']['extbase_object']['backend'] = 'TYPO3\\CMS\\Core\\Cache\\Backend\\FileBackend';
}
```
**Hinweis:**

Wenn Sie bereits eine AdditionalConfiguration.php nutzen, ergänzen Sie bitte die oben genannten Einträge.
Achten Sie bitte darauf, dass keine Windows-Steuerzeichen in der Datei vorhanden sind. Dies passiert zum Beispiel, wenn Sie die Datei unter Windows mit Notepad erstellen und anschließend hochladen.

Ein sicherer Weg zur Erstellung der Datei und der notwendigen Einträge ohne Steuerzeichen bietet zum Beispiel der WebFTP-Zugang im Mittwald Kundencenter.

Alternativ können Sie die Datei lokal mit einem Editor erstellen, der Unix-konform speichern kann. Dies ist zum Beispiel mit Notepad++ oder UltraEdit möglich. 

---

# APC with TYPO3: high fragmentation over time
Quelle: https://stackoverflow.com/questions/24116939/apc-with-typo3-high-fragmentation-over-time - 2014-06-24

### Question:
Using APCu with TYPO3 6.2 extensively, I always get a high fragmentation of the cache over time. I already had values of 99% with a smaller shm_size.

In case you are a TYPO3 admin, I also switched the caches `cache_pagesection`, `cache_hash`, `cache_pages` (currently for testing purposes moved to DB again), `cache_rootline`, `extbase_reflection`, `extbase_opject` as well as some other extension caches to apc backend. Mainly switching the `cache_hash` away from DB sped up menu rendering times dramatically (https://forge.typo3.org/issues/57953)

1. Does APC fragmentation matter at all or should I simply watch out that it just never runs out of memory?
2. To TYPO3 admins: do you happen to have an idea which tables cause fragmentation most and what bit in the apcu.ini configuration is relevant for usage with TYPO3?
>>>
I already tried using `apc.stat = 0`, `apc.user_ttl = 0`, `apc.ttl = 0` (as in the T3 caching guide http://docs.typo3.org/typo3cms/CoreApiReference/CachingFramework/FrontendsBackends/Index.html#caching-backend-apc) and to increase the `shm_size` (currently at 512M where normally around 100M would be used). Shm_size does a good job at reducing fragmentation, but I'd rather have a smaller but full cache than a large one unused.
>>>

3. To APC(u) admins: could it be that frequently updating cache entries that change in size as well cause most of the fragmentation? Or is there any other misconfiguration that I'm unaware of?
>>>
I know there is a lot of entries in cache (mainly JSON data from remote servers) where some of them update every 5 minutes and normally are a different size each time. If that is indeed a cause, how can I avoid it? Btw: APCU Info shows there are a lot of entries taking up only 2kB but each with a fragmented spacing of about 200 Bytes.
>>>

4. To TYPO3 and APC admins: apc has a great integration in TYPO3, but for more frequently updating and many small entries, would you advise a different cache backend than apc?

### (self) Answer:
This is no longer relevant for us, I found a different solution reverting back to MySQL cache. Though if anyone comes here via search, this is how we did it in the end:

Leave the APC cache alone and only use it for the preconfigured `extbase_object` cache. This one is less than 1MB, has only a few inserts at the beginning and yields a very high hit / miss ratio after. As stated in the install tool in the section "Configuration Presets", this is what the cache backend has been designed for.

I discovered this bug https://forge.typo3.org/issues/59587 in the process and reviewed our cache usage again. It resulted in huge cache entries only used for tag-to-ident-mappings. My conclusion is, even after trying out the fixed cache, that APCu is great for storing frequently accessed key-value mappings but yields when a lot of frequently inserted or tagged entries are around (such as `cache_hash` or `cache_pages`).

Right now, the MySQL cache tables have a better performance with extended usage of the MySQL server memory cache (but in contrast to APCu with disc backup). This was the magic setup for our my.cnf (found here: http://www.mysqlperformanceblog.com/2007/11/01/innodb-performance-optimization-basics/):
```
innodb_buffer_pool_size = 512M
innodb_log_file_size = 256M
innodb_log_buffer_size = 8M
innodb_flush_log_at_trx_commit = 2
innodb_thread_concurrency = 8
innodb_flush_method=O_DIRECT
innodb_file_per_table
```
With this additional MySQL server setup, the default typo3 cache tables do their job best.

---

# Redis als Cache-Backend in TYPO3
Quelle: https://blog.herrhansen.com/redis-cache-typo3/ - 2013-01-30

Das Caching-Framework bietet seit TYPO3 4.5 viele Möglichkeiten das Caching zu optimieren.

Eine, wie ich finde, sehr gute Möglichkeit, gerade wenn man mehrere TYPO3-Installationen auf einem Server versammelt sind, ist die Möglichkeit Teile des Cache in Redis abzulegen.

Durch die Möglichkeit in Redis viele verschiedene Datenbanken anzulegen werden die Caches sauber voneinander getrennt und man kann die Vorteile einer RAM-basierten Datenbank zu nutzen. Dies ist für mich ein entscheidender Vorteil gegenüber Memcache und lässt sich für die verschiedenen Cache-Backends nutzen.

Für TYPO3 < 6.2:
```php
$TYPO3_CONF_VARS[‚SYS‘][‚caching‘][‚cacheConfigurations‘][‚cache_pages‘] = array(
    ‚backend‘ => ‚t3lib_cache_backend_RedisBackend‘,
    ‚options‘ => array(
        ‚defaultLifetime‘ => 86400,
        ‚database‘ => 0
        )
    );
$TYPO3_CONF_VARS[‚SYS‘][‚caching‘][‚cacheConfigurations‘][‚cache_pagesection‘] = array(
    ‚backend‘ => ‚t3lib_cache_backend_RedisBackend‘,
    ‚options‘ => array(
        ‚defaultLifetime‘ => 86400,
        ‚database‘ => 1
        )
    );
$TYPO3_CONF_VARS[‚SYS‘][‚caching‘][‚cacheConfigurations‘][‚cache_hash‘] = array(
    ‚backend‘ => ‚t3lib_cache_backend_RedisBackend‘,
    ‚options‘ => array(
        ‚defaultLifetime‘ => 86400,
        ‚database‘ => 2
        )
    );
$TYPO3_CONF_VARS[‚SYS‘][‚caching‘][‚cacheConfigurations‘][‚extbase_object‘] = array(
    ‚backend‘ => ‚t3lib_cache_backend_RedisBackend‘,
    ‚options‘ => array(
        ‚defaultLifetime‘ => 86400,
        ‚database‘ => 3
        )
    );
$TYPO3_CONF_VARS[‚SYS‘][‚caching‘][‚cacheConfigurations‘][‚extbase_reflection‘] = array(
    ‚backend‘ => ‚t3lib_cache_backend_RedisBackend‘,
    ‚options‘ => array(
        ‚defaultLifetime‘ => 86400,
        ‚database‘ => 4
        )
    );
```
Für TYPO3 >= 6.2 sieht das dann folgendermaßen aus:
```php
$TYPO3_CONF_VARS['SYS']['caching']['cacheConfigurations']['cache_pages']['backend'] = 'TYPO3\\CMS\\Core\\Cache\\Backend\\RedisBackend',
$TYPO3_CONF_VARS['SYS']['caching']['cacheConfigurations']['cache_pages']['options'] = array(
        'defaultLifetime' => 86400,
        'database' => 0
        );
$TYPO3_CONF_VARS['SYS']['caching']['cacheConfigurations']['cache_pagesection']['backend'] = 'TYPO3\\CMS\\Core\\Cache\\Backend\\RedisBackend',
$TYPO3_CONF_VARS['SYS']['caching']['cacheConfigurations']['cache_pagesection']['options'] = array(
        'defaultLifetime' => 86400,
        'database' => 1
        );
$TYPO3_CONF_VARS['SYS']['caching']['cacheConfigurations']['cache_hash']['backend'] = 'TYPO3\\CMS\\Core\\Cache\\Backend\\RedisBackend',
$TYPO3_CONF_VARS['SYS']['caching']['cacheConfigurations']['cache_hash']['options'] = array(
        'defaultLifetime' => 86400,
        'database' => 2
        );
$TYPO3_CONF_VARS['SYS']['caching']['cacheConfigurations']['extbase_object']['backend'] = 'TYPO3\\CMS\\Core\\Cache\\Backend\\RedisBackend',
$TYPO3_CONF_VARS['SYS']['caching']['cacheConfigurations']['extbase_object']['options'] = array(
        'defaultLifetime' => 86400,
        'database' => 3
        );
$TYPO3_CONF_VARS['SYS']['caching']['cacheConfigurations']['extbase_reflection']['backend'] = 'TYPO3\\CMS\\Core\\Cache\\Backend\\RedisBackend',
$TYPO3_CONF_VARS['SYS']['caching']['cacheConfigurations']['extbase_reflection']['options'] = array(
        'defaultLifetime' => 86400,
        'database' => 4
        );
```
oder zum direkten Einfügen in das caching-Array in der LocalConfiguration.php:
```php
'cacheConfigurations' => array(
    'cache_pages' => array(
        'backend' => 'TYPO3\\CMS\\Core\\Cache\\Backend\\RedisBackend',
        'options' => array(
            'defaultLifetime' => 86400,
            'database' => 0
        )
    ),
    'cache_pagesection' => array(
        'backend' => 'TYPO3\\CMS\\Core\\Cache\\Backend\\RedisBackend',
        'options' => array(
            'defaultLifetime' => 86400,
            'database' => 1
        )
    ),        
    'cache_hash' => array(
        'backend' => 'TYPO3\\CMS\\Core\\Cache\\Backend\\RedisBackend',
        'options' => array(
            'defaultLifetime' => 86400,
            'database' => 2
        )
    ),        
    'extbase_object' => array(
        'backend' => 'TYPO3\\CMS\\Core\\Cache\\Backend\\RedisBackend',
        'options' => array(
            'defaultLifetime' => 86400,
            'database' => 3
        )
    ),        
    'extbase_reflection' => array(
        'backend' => 'TYPO3\\CMS\\Core\\Cache\\Backend\\RedisBackend',
        'options' => array(
            'defaultLifetime' => 86400,
            'database' => 4
        )                                                                                                                             
    )
)
```
In dieser Beispielkonfiguration wurde die Lebensdauer für alle Caches exemplarisch auf 24h gestellt, das kann man natürlich für die eigenen Bedürfnisse noch optimieren.

Das Redis Backend hat in meinem Fall die Auslierung der gecachten Seiten noch einmal um über 100ms beschleunigt. Klingt nach nicht sehr viel, bedeutet aber in meinem Fall einen Performanceschub um über 30%.

http://redis.io
http://docs.typo3.org/typo3cms/CoreApiReference/CachingFramework/Configuration/Index.html

### Anmerkung aus den Comments:
Für Typo3 6.2+ klappt der Aufruf nicht mehr, folgende syntax ist notwendig:
```php
$GLOBALS['TYPO3_CONF_VARS']['SYS']['caching']['cacheConfigurations']['cache_pages']['backend'] = 'TYPO3\\CMS\\Core\\Cache\\Backend\\RedisBackend';
$GLOBALS['TYPO3_CONF_VARS']['SYS']['caching']['cacheConfigurations']['cache_pages']['options'] = array(
'database' => 3,
);
```
siehe auch: https://gist.github.com/cedricziel/5744800
---

