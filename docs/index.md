# Home

For the source code visit the [GitHub Repository](https://github.com/materialsproject/MPCite).

For package instructions visit [PyPi](https://pypi.org/p/mp-cite).

# What is MPCite?

MPCite provides high-throughput, orchestrated functions for the Materials Project to interact programmatically with the DOE Office of Scientific and Technical Information (OSTI) via the E-Link API [[1]](https://github.com/doecode/elinkapi). It enables automated submission, validation, and management of metadata records and Digital Object Identifiers (DOIs) for materials data within the Materials Project ecosystem.

# What is E-Link?

Elink is 

# Implementation

MPCite's core functionalities are python functions which can be imported from the package. Altogether, they perform the afforementioned workflow.

## First, identify DOI entries to change...

```python
def find_out_of_date_doi_entries(
    rc_client: MongoClient,
    doi_client: MongoClient,
    robocrys_db: str,
    robocrys_collection: str,
    doi_db: str,
    doi_collection: str,
) -> list[OstiID]
```

## Next, update said DOI entries...

```python
def update_existing_osti_record(
    elinkapi: Elink, osti_id: OstiID, new_values: dict
) -> RecordResponse
```

```python
def update_state_of_osti_record(
    elinkapi: Elink, osti_id: OstiID, new_state="submit"
) -> RecordResponse
```

## If there are new records to be submitted...

```python
def submit_new_osti_record(
    elinkapi: Elink,
    new_record: Record,
    state="submit",
) -> RecordResponse
```

## And miscellaneous functions...

```python
def make_minimum_record_to_fully_release(
    title,  # required to make record
    product_type="DA",  # required to make record
    organizations=[
        Organization(type="RESEARCHING", name="LBNL Materials Project (LBNL-MP)"),
        Organization(
            type="SPONSOR",
            name="TEST SPONSOR ORG",
            identifiers=[{"type": "CN_DOE", "value": "AC02-05CH11231"}],
        ),
    ],  # sponsor org is necessary for submission
    persons=[Person(type="AUTHOR", last_name="Perrson")],
    site_ownership_code="LBNL-MP",
    access_limitations=["UNL"],
    publication_date=datetime.now().replace(
        hour=0, minute=0, second=0, microsecond=0
    ),  # what should this be?
    site_url="https://next-gen.materialsproject.org/materials",
) -> Record
```

One can delete OSIT records with a call to

```python
def delete_osti_record(
    elinkapi_token: str, osti_id: OstiID, reason: str
) -> RecordResponse
```

And additionally, there is a function to empty the review environment, which is handy for development and debugging.

```python
def emptyReviewAPI(reason, review_api)
```

# Data Structures

A new DOI document format was made in preparation to upgrade the current DOI collection in `mp_core`.

## DOI Builder Model

```
doi_builder.py
A doi collection must store the following information about a document:
- doi number
- title
- osti id (ELink's Unique Identifier)
- material id (MP's Unique Identifier)
- date of system entry date (Date (UTC) of this revision's inception)
- date of last update (date edited or date_submitted_to_osti_last) (take from ELink)
- workflow status and the date (?) of each step:
    - SA, saved, in a holding state, not to be processed
    - SR, submit to releasing official "released_to_osti_date, as entered by releasing official"
    - SO, submit to OSTI 
    - SF, submitted but failed validation
    - SX, submitted but failed to release
    - SV, submitted and validated
    - R, released
```

## Record Response

```
Here is an example of RecordResponse
RecordResponse(
    osti_id=2523296,
    workflow_status='SA',
    access_limitations=['UNL'],
    access_limitation_other=None,
    announcement_codes=None,
    availability=None,
    edition=None,
    volume=None,

    # Identifiers
    identifiers=[
        Identifier(type='CN_NONDOE', value='EDCBEE'),
        Identifier(type='CN_DOE', value='AC02-05CH11231'),
        Identifier(type='RN', value='mp-1037659'),
    ],

    # People involved
    persons=[
        Person(
            type='CONTACT',
            first_name='Kristin',
            last_name='Persson',
            phone='+1(510)486-7218',
            email=['feedback@materialsproject.org'],
            affiliations=[
                Affiliation(name='LBNL')
            ]
        )
    ],

    # Organizations
    organizations=[
        Organization(name='The Materials Project', type='CONTRIBUTING', contributor_type='ResearchGroup'),
        Organization(name='LBNL Materials Project', type='RESEARCHING'),
        Organization(name='Lawrence Berkeley National Laboratory (LBNL), Berkeley, CA (United States)', type='RESEARCHING'),
        Organization(name='USDOE Office of Science (SC), Basic Energy Sciences (BES) (SC-22)', type='SPONSOR'),
        Organization(name='MIT', type='CONTRIBUTING', contributor_type='Other'),
        Organization(name='UC Berkeley', type='CONTRIBUTING', contributor_type='Other'),
        Organization(name='Duke', type='CONTRIBUTING', contributor_type='Other'),
        Organization(name='U Louvain', type='CONTRIBUTING', contributor_type='Other'),
    ],

    # Metadata
    country_publication_code='US',
    doe_supported_flag=False,
    doi='10.17188/1714845',
    edit_reason='Record updated upon request of LBNL-MP to remove authors and replace with a single collaborator.',
    format_information='',
    invention_disclosure_flag=None,
    paper_flag=False,
    peer_reviewed_flag=False,
    product_type='DA',
    publication_date=datetime.date(2020, 4, 30),
    publication_date_text='04/30/2020',
    site_url='https://materialsproject.org/materials/mp-1037659',
    site_ownership_code='LBNL-MP',
    site_unique_id='mp-1037659',
    subject_category_code=['36'],
    title='Materials Data on RbYMg30O32 by Materials Project',

    # Description
    description="""
        RbMg₃₀YO₃₂ is Molybdenum Carbide MAX Phase-derived and crystallizes in the tetragonal P4/mmm space group.
        Rb¹⁺ is bonded to six O²⁻ atoms to form RbO₆ octahedra...
        (Truncated here for brevity, full description is included in original)
    """,

    keywords=['crystal structure', 'RbYMg30O32', 'Mg-O-Rb-Y'],
    languages=['English'],
    related_doc_info='https://materialsproject.org/citing',

    # Media
    media=[
        MediaInfo(
            media_id=1908478,
            osti_id=2523296,
            status='C',
            mime_type='text/html',
            files=[
                MediaFile(
                    media_file_id=12017281,
                    media_type='O',
                    url='https://materialsproject.org/materials/mp-1037659'
                ),
                MediaFile(
                    media_file_id=12017284,
                    media_type='C',
                    mime_type='text/html',
                    media_source='OFF_SITE_DOWNLOAD'
                )
            ]
        )
    ],

    # Audit logs
    audit_logs=[
        AuditLog(
            messages=['Revision status is not correct, found SA'],
            status='FAIL',
            type='RELEASER',
            audit_date=datetime.datetime(2025, 6, 30, 22, 30, 24, 865000, tzinfo=TzInfo(UTC))
        )
    ],

    # Timestamps
    date_metadata_added=datetime.datetime(2025, 6, 30, 22, 30, 20, 495000, tzinfo=TzInfo(UTC)),
    date_metadata_updated=datetime.datetime(2025, 6, 30, 22, 30, 22, 247000, tzinfo=TzInfo(UTC)),

    # Misc
    revision=2,
    added_by=139001,
    edited_by=139001,
    collection_type='DOE_LAB',
    hidden_flag=False
)
```