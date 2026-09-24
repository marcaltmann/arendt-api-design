# Design Arendt-API

## Person-API

### Person-API
- *TEI-XML Persons? Wo werden die verwaltet? Sind sie die tatsächliche Quelle?*
- alle Details können in die Liste, im Moment bei 700 Einträgen 16KB gzipped!
- Reihenfolge der Keys ist in Liste und Detail unterschiedlich.
- href/link-Key wäre besser als api-Key in person-List?
- label: schwierig maschinell zu verarbeiten bei [de] [en], auch *inkonsistent* (manchmal de vorne, manchmal en) -> d.h. man würde ohnehin direkt auf persName.regular.de zugreifen.
- Es werden oft Keys aus den TEI-XML-Bezeichnern durchgeschleift, z.B. personGrp,
  regular, alternative, role, persName usw.
- role == null sollte "historical" sein
- Mögliche role-Werte ändern sich je nach type
- Kombinationen nicht möglich, z.B. mythological dynasty
- im Moment folgende role-Werte:
- type: "person" — null, "fictitious", "mythological", "biblical", "deity"
- type: "personGrp" — "siblings", "family", "spouses", "dynasty"

- Alternative:
  ```json
  [
    { "type": "person",      "status": "historical",   "groupType": null },
    { "type": "person",      "status": "fictional",    "groupType": null },
    { "type": "personGroup", "status": "historical",   "groupType": "siblings" },
    { "type": "personGroup", "status": "mythological", "groupType": "dynasty" }
  ]
  ```

- identifiers:
  ```json
  [
    {
      "type/scheme/provider": "gnd",
      "id": "118680315",
      "url": "https://d-nb.info/gnd/118680315"
    }
  ]
  ```

- Datumsangaben: Besser EDTF Level 1 einführen bzw. vereinbaren (in OpenAPI-Beschreibung),
  aber es muss auch entsprechend in den TEI-XMLs ausgezeichnet werden (@when-custom statt @when-iso,
  geht offenbar auch auf demselben Element)
- weshalb? Jahres oder Monatsangaben, oder Ausdrücken von Unsicherheit (ca.-Werte)

Gesamt-Vorschlag:
```json
{
  "id": "ae6870711",
  "type": "person",
  "status": "historical",
  "groupType": null,
  "name": {
    "preferred": {
      "de": "Marx, Karl",
      "en": "Marx, Karl"
    },
    "variants": [
      {
        "de": "Marx, Carl",
        "en": "Marx, Carl"
      }
    ]
  },
  "birth": "1818-05-05",
  "death": "1883-03-14",
  "relations": [],
  "identifiers": [
    {
      "scheme": "gnd",
      "id": "118578537",
      "url": "https://d-nb.info/gnd/118578537"
    }
  ],
  "links": {
    "self": "/api/v0/index/persons/ae6870711"
  }
}
```

## API generell / Content Negotation

- Hypermedia-API geplant?
- Pagination geplant? Jetzt schon "total"-Key einbauen
- JSON für TEI-XML nicht sehr sinnvoll
- Content Negotiation alleine hat Nachteile, nicht per E-Mail zu versenden etc.
- Muss aber vorhanden sein, weil Content Negotiation für den gesamten Endpunkt
  gelten muss.
- Deshalb ganz anderer Vorschlag: unterschiedliche Endpunkte, weil es sich doch
  um unterschiedliche Ressourcen handelt:
  - /texts/resources/ae1254820 -> JSON oder JSON und XML
  - /texts/resources/ae1254820/tei -> nur XML
  - /texts/resources/ae1254820/plain -> nur text/plain
- Dann evtl. im Sinne von Hypermedia-API:
  ```json
  {
    "links": {
      "self": "/api/v0/texts/resources/ae1254820",
      "tei": "/api/v0/texts/resources/ae1254820/tei",
      "plain": "/api/v0/texts/resources/ae1254820/plain"
    }
  }
  ```

## SwaggerUI

- title-Tag immer noch "API Documentation"
- Description of Param nicht benutzerfreundlich, besser Beispiel: ID of a person (pattern: ^ae\d{7}$)
- Es fehlt eine Schemas-Sektion. Durch sie könnte man vor allem klarer dokumentieren,
welche möglichen Werte es für einen bestimmten Key gibt, z.B. für Persons in der person-list.
- Wegen Performance-Problemen bei SwaggerUI bei großen XML-Dateien syntaxHighlight ausstellen:
  ```js
	const defaultOptions = {
		displayOperationId: true,
		displayRequestDuration: true,
		showExtensions: true,
		withCredentials: true,
		syntaxHighlight: false
	}
  ```


## Resource-API

### Generelle API-Design-Guidelines

- JSON-Keys: CamelCase, ausgeschriebene Wörter (nicht abgekürzt)
- an Schema.org orientieren?
- Unabhängigkeit von TEI-XML
- Wenn Wert nicht vorhanden, null statt Weglassen des JSON-Keys. D.h.
  alle JSONs haben dieselbe Anzahl von Keys (was ist bei Arrays, leer oder null?)

### Vorschläge für Resource-JSONs

#### Sprache auszeichnen
- Sofort: language (String) oder languages (Array)
- Später: otherLanguages (Array)
- Format: BCP 47 wie in TEI-XML

#### Eine Sprache offenbar falsch kodiert
- [BCP47 language subtag lookup](https://r12a.github.io/app-subtags/)
- grc-la -> Altgriechisch, wie in Laos verwendet
- grc-Latn -> Altgriechisch in lateinischer Schrift
- -> überall falsch kodiert, wo es angewendet wird. 44 Dateien insgesamt.

#### Versions / Beispiel Ideologie und Terror

- offenbar im Moment nur einmal verwendet, bei Ideologie und Terror Typoskript (ae3016015)
- Ids: "ae3016015_l" und "ae3016015_s" inkonsistent, andere Lösung finden
- Ideologie und Terror:
  - ae3016015 -> Typoskript mit den beiden Versionen ae3016015_l und ae3016015_s
  - ae0246860 -> Typoskript rekonstruierte Originalfassung
  - ae3572513 -> Band 6, 11-25 Fassung 1 deutsch
  - ae7441502 -> Band 6, 26-51 Fassung 2 deutsch
  - englische Fassungen nicht über versions-Key verknüpft:
    - ae7967323 -> Proto-Ideology and Terror Typoskript
    - ae8560990 -> Ideology and Terror (Band 6, Seiten I-J) -> falsche Seitenangaben!


#### Type
- im Moment: type: editorial oder source.print, source.typescript -> wieder Vermischung
  von zwei verschiedenen Sachen
- ist hier gemeint: type: primary/editorial
- print/typescript könnte innerhalb von 'source' als 'form' stehen
- unter source stehen zwei verschiedene Sätze von JSON-Keys, je nachdem ob es
  source.print oder source.typescript ist, bei editorial steht null

#### Vorschläge:

Auszüge...

```
{
  "id": "ae1141775",
  "type": "primary",
  "source": {
    "form": "print",
    "publication": {
      "authors": [
        "Hannah Arendt"
      ],
      "title": {
        "analytic": "Sechs Essays",
        "series": "Schriften der Wandlung"
      },
      "editors": [
        "Dolf Sternberger"
      ],
      "pubPlace": "Heidelberg",
      "date": "1948",
      "biblScope": {
        "volume": "3",
        "page": {
          "from": "5",
          "to": "10"
        }
      }
    },
    "holding": null
  }
}
```

und

```
{
  "id": "ae0565102",
  "type": "primary",
  "source": {
    "form": "typescript",
    "publication": null,
    "holding": {
      "repository": "Deutsches Literaturarchiv Marbach",
      "collection": null,
      "shelfmark": null,
      "uri": null,
      "physicalLocation": null
    }
  }
}

```

#### Typ der Ressource auszeichnen -> noch unklar

- Welchen Typ beschreibt das JSON-Dokument, z.B. Werk, Textversion, vielleicht nach diesem
  WEMI-System?
- Ich weiß nicht, auf welcher Ebene ich mich gerade befinde.

#### Neue Entität "work"?

- Überlegen, ob man übergreifende Werk-Kategorie einführt, die man mit Normdaten (GND-Id,
  Wikidata-Id) besser verknüpfen kann.
