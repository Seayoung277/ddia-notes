# Linkedin Profile Data Model Example

## Data Model Design with Relational Model

![Data model design with relational model](./linkedin-profile-exp-img-1.jpeg)

Note that for position/education/contact information, data is normalized with extra tables and foreign keys

To fetch the whole profile, you need to either perform multiple queries, or perform a multi-way join query

## Data Model Design with Document Model

![Data model design with document model](./linkedin-profile-exp-img-2.jpeg)

Note that for user/region/industry, ID is used instead of localized data. Most document databases do not support join, many-to-one/many-to-many relations like this might need to be handled by application codes.