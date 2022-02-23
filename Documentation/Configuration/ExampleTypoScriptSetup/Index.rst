.. ==================================================
.. FOR YOUR INFORMATION
.. --------------------------------------------------
.. -*- coding: utf-8 -*- with BOM.

.. include:: ../../Includes.txt

Example TypoScript Setup
^^^^^^^^^^^^^^^^^^^^^^^^

The following example shows all usable settings for the extension:

::

    plugin.tx_dpnglossary {
        settings {
            pagination {
                characters = A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z
                insertAbove = 1
                insertBelow = 0
            }

            termWraps = CASE
            termWraps {
                key.field = term_type
                default = TEXT
                default {
                    field = name
                    dataWrap = |
                    typolink {
                        ATagParams.dataWrap = title="{field:tooltiptext}" class="dpnglossary link"
                        ATagParams.dataWrap {
                            override = title="{field:name}" class="dpnglossary link"
                            override.if.isFalse.data = field:tooltiptext
                        }
                        useCacheHash = 1
                    }
                }

                abbreviation {
                    dataWrap = <abbr title="{field:tooltiptext}" lang="{field:term_lang}">|</abbr>
                    dataWrap {
                        override = <abbr title="{field:name}" lang="{field:term_lang}">|</abbr>
                        override.if.isFalse.data = field:tooltiptext
                    }
                }

                acronym {
                    dataWrap = <acronym title="{field:tooltiptext}" lang="{field:term_lang}">|</acronym>
                    dataWrap {
                        override = <acronym title="{field:name}" lang="{field:term_lang}">|</acronym>
                        override.if.isFalse.data = field:tooltiptext
                    }
                }

                definition {
                    dataWrap = <dfn title="{field:tooltiptext}" lang="{field:term_lang}">|</dfn>
                    dataWrap {
                        override = <dfn title="{field:name}" lang="{field:term_lang}">|</dfn>
                        override.if.isFalse.data = field:tooltiptext
                    }
                }
            }
        }
    }

Configure Routing for terms and pagination
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  DpnGlossary:
    type: Extbase
    limitToPages: [YOUR_PLUGINPAGE_UID]
    extension: DpnGlossary
    plugin: glossary
    routes:
    - { routePath: '/{character}', _controller: 'Term::list', _arguments: {'character': 'currentCharacter'} }
    - { routePath: '/{localized_term}/{term_name}', _controller: 'Term::show', _arguments: {'term_name': 'term'} }
    defaultController: 'Term::list'
    defaults:
      character: ''
    aspects:
      term_name:
        type: PersistedAliasMapper
        tableName: 'tx_dpnglossary_domain_model_term'
        routeFieldName: 'url_segment'
      character:
        type: StaticMultiRangeMapper
        ranges:
          - start: 'A'
            end: 'Z'
      localized_term:
        type: LocaleModifier
        default: 'term'
        localeMap:
        - locale: 'de_DE.*'
          value: 'begriff'

Configure full url preview for the term slug field
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Setting the detailpage uid in the tsconfig will enable the full url preview for terms slug fields:

::

  TCEFORM {
    tx_dpnglossary_domain_model_term {
      url_segment.config.previewUrl.pageUid = [YOUR_PLUGINPAGE_UID]
    }
  }

Create XML Sitemap for Terms
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Requirements: System extension "SEO" installed (and setup.typoscript added)

Add the following code to you setup:

.. code-block:: typoscript

  plugin.tx_seo.config {
      xmlSitemap {
          sitemaps {
              glossar {
                  provider = TYPO3\CMS\Seo\XmlSitemap\RecordsXmlSitemapDataProvider
                  config {
                      table = tx_dpnglossary_domain_model_term
                      additionalWhere =
                      sortField = uid
                      lastModifiedField = tstamp
                      pid = 123 #uid of the sysfolder where your term are stored
                      recursive = 2
                      url {
                          pageId = 456 #uid of the page where the glossary plugin is placed
                          fieldToParameterMap {
                              uid = tx_dpnglossary_glossary[term]
                          }
                          additionalGetParameters {
                              tx_dpnglossary_glossary.controller = Term
                              tx_dpnglossary_glossary.action = show
                          }
                          useCacheHash = 1
                      }
                  }
              }
          }
      }
  }

Official documentaton: `https://docs.typo3.org/m/typo3/reference-coreapi/10.4/en-us/ApiOverview/XmlSitemap/Index.html`
