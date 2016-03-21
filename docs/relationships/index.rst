The Relationships Module
========================

This module uses Wagtail's implementation of ``django-taggit`` to automatically create relationships between individual
pieces of content. The relationship between any two pieces of content is assigned a numerical score (decimal) between
0 and 1, based on configurable scoring factors. Any two pieces of content are considered to be "related" if they
share one or more tags.

Whereas tagging represents a flat taxonomy, the relationships module includes the ability to categorize tags into
a hierarchical taxonomy through ``django-treebeard``. Categories are currently managed through ``django.contrib.admin``.

Module Installation
-------------------
First, add the module to ``settings.INSTALLED_APPS``::

    INSTALLED_APPS = (
        ...
        'wagtailplus.wagtailrelations',
        ...
    )

If ``treebeard`` isn't already in your ``INSTALLED_APPS``, you'll need to add that as well.
Then run ``manage.py syncdb`` (Django < 1.7) or ``manage.py migrate`` (Django >= 1.7).

Settings
^^^^^^^^
The following settings my
be added to ``settings.py`` to influence the score assigned to two pieces of content::

    AUTHORITATIVE_FACTOR    = 0.25
    CATEGORY_FACTOR         = 0.25
    LIKE_TYPE_FACTOR        = 0.25
    TAG_FACTOR              = 0.25

Added Methods and Properties
----------------------------
All model classes that inherit from ``wagtail.wagtailcore.models.Page`` and include a "tags" field will automatically
include ``get_related()`` and ``get_related_with_scores()`` methods, in addition to ``related`` and
``related_with_scores`` properties.

Related
^^^^^^^
``get_related()`` and ``related`` return a list of related instances that include
both a "title" and "url" attribute::

    for related in my_page_instance.get_related():
        print related.title, related.url

Related with Scores
^^^^^^^^^^^^^^^^^^^
``get_related_with_scores()`` and ``related_with_scores`` return a list of tuples containing the score and related
instance (sorted by high score)::

    for score, related in my_page_instance.get_related_with_scores().iteritems():
        print score, related.title, related.url

Scoring Factors
---------------
The relationships module utilizes four configurable factors in assigning a numerical score between any two pieces
of related content, allowing individual implementations to weight scoring according to their needs. These factors are
normalized so that the highest possible score is 1.0.

Authoritative Score
^^^^^^^^^^^^^^^^^^^
The authoritative score is calculated by dividing the length of time between the creation and last modification of a
piece of content by its age. Content that is updated as it ages receives a higher score than newer content or older
content that hasn't been updated.

Category Score
^^^^^^^^^^^^^^
The category score is calculated by adding the ratio of category depth to total number of items assigned to category
for each category that two pieces of content have in common, then dividing that sum by the total number of categories
that the two pieces of content have in common. The more specific (deeper) the category, the higher the score.

Like Type Score
^^^^^^^^^^^^^^^
The like type score is calculated by dividing the number of inherited classes that two pieces of content have in common
by the total number of inherited classes between the two pieces of content. Similar content types will score higher
than dissimilar types.

Tag Score
^^^^^^^^^
The tag score is calculated by dividing the number of tags that two pieces of content have in common by the total
number of tags assigned to the two pieces of content.
