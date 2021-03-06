# API Guide

Detailed explanation and examples to using Edvisor Api

## Accessing the price of a course

> The GraphQL query could look like this:

```
query {
  # lookup a specific offering
  offering(offeringId: 1) {
    # get the offeringCourse relationship
    offeringCourse {
      # get the prices specified by the filter
      prices(filter: {
        startDate: {
          eq: "2018-01-01"
        },
        durationTypeId: 3,
        durationAmount: {
          eq: 8
        },
        nationalityId: 29,
        age: {
          eq: 21
        },
        agencyCountryId: 29
      }) {
        # specify the fields we want to see returned
        durationAmount,
        durationType {
          codeName
        },
        startDate,
        currency {
          code
        },
        originalPrice,
        hasPromotion
      }
    }
  }
}
```

> You can copy and paste this cURL command to test it out!

```shell
curl 'https://api.edvisor.io/graphql' \
  -H 'Authorization: Bearer <your_edvisor_api_key>' \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":"query {\n  offering(offeringId: 417) {\n    offeringCourse {\n      prices(filter: {\n        startDate: {\n          eq: \"2018-01-01\"\n        }\n      }) {\n        durationAmount,\n        durationType {\n          codeName\n        },\n        startDate,\n        currency {\n          code\n        },\n        originalPrice,\n        hasPromotion\n      }\n    }\n  }\n}","variables":null}' \
  --compressed
```

A very common usage of the Edvisor API is to query for the pricing of a course.

This can be achieved easily with the `prices`</a> property on the <a href='http://docs.edvisor.io/schema/offeringcourse.doc.html'>`OfferingCourse`</a> object. 

You can read more about the parameters that the `prices` property accepts here: <a href='http://docs.edvisor.io/schema/offeringcourse.doc.html'>http://docs.edvisor.io/schema/offeringcourse.doc.html</a>

In the example to the left, we are using the root query `offering` to find a specific offering, then traversing through the `offeringCourse` relation to get to the `prices` property. 

You will notice that the `prices` property takes in a `filter` parameter which you can customize to meet your needs. You will notice that the `prices` property will return a list of `DerivedPrice` objects, which is described here: <a href='http://docs.edvisor.io/schema/derivedprice.doc.html'>http://docs.edvisor.io/schema/derivedprice.doc.html</a>

## Performing a search for courses

> The GraphQL query could look like this:

```
query {
  # use the offeringSearch query
  offeringSearch(filter: {
    # filter for only Courses (offeringTypeId = 1)
    offeringTypeIds: [1],
    durationTypeId: 3,
    durationAmount: {
      eq: 4
    },
    pagination: {
      limit: 10
    }
  }) {
    ... on OfferingCourseSearchResult {
      offering {
        name,
        school {
          name,
          schoolCompany {
            name
          }
        }
      }
    	offeringId,
      durationAmount,
      durationType {
        codeName
      },
      price {
        originalPrice,
        currency {
          code
        }
      }
  	}
  }
}
```

> You can copy and paste this cURL command to test it out!

```shell
curl 'https://api.edvisor.io/graphql' \
  -H 'Authorization: Bearer <your_edvisor_api_key>' \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":"query {\n  offeringSearch(filter: {\n    offeringTypeIds: [1],\n    durationTypeId: 3,\n    durationAmount: {\n      eq: 4\n    },\n    pagination: {\n      limit: 10\n    }\n  }) {\n    ... on OfferingCourseSearchResult {\n      offering {\n        name,\n        school {\n          name,\n          schoolCompany {\n            name\n          }\n        }\n      }\n    \tofferingId,\n      durationAmount,\n      durationType {\n        codeName\n      },\n      price {\n        originalPrice,\n        currency {\n          code\n        }\n      }\n      \n  \t}\n  }\n}","variables":null}' \
  --compressed
```

Our API currently exposes the ability to search for offerings matching specific search criteria. 

You can use the `offeringSearch` query to perform a search for matching offerings, and also fetch the prices! For more details on what criteria you can set in the filter, please visit our documentation on the `OfferingSearchResultFilter` here: <a href='http://docs.edvisor.io/schema/offeringsearchresultfilter.doc.html'>http://docs.edvisor.io/schema/offeringsearchresultfilter.doc.html</a>. For more details on the result objects returned, please visit our documentation on the `OfferingSearchResult` object here: <a href='http://docs.edvisor.io/schema/offeringsearchresult.doc.html'>http://docs.edvisor.io/schema/offeringsearchresult.doc.html</a>

## Exploring your connected Schools

> The GraphQL query could look like this:

```
query {
  # start by querying for your own AgencyCompany object
  agencyCompany {
    agencyCompanyId,
    name,
    # now fetch the connectedSchoolCompanies relation
    connectedSchoolCompanies {
      schoolCompanyId,
      name,
      schools {
        name,
        # here we are fetching all the courses related to your schools
        courses {
          offeringId,
          name
        }
      }
    }
  }
}
```

> You can copy and paste this cURL command to test it out!

```shell
curl 'https://api.edvisor.io/graphql' 
  -H 'Authorization: Bearer <your_edvisor_api_key>' \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":"query {\n  agencyCompany {\n    agencyCompanyId,\n    name,\n    connectedSchoolCompanies {\n      schoolCompanyId,\n      name,\n      schools {\n        name,\n        courses {\n          offeringId,\n          name\n        }\n      }\n    }\n  }\n}","variables":null}' 
  --compressed 
```

If you are an agency customer, you have full ability to explore the data of the schools you're connected to through our API.

To begin, we recommend you start with the `agencyCompany` query which will return to you data on your own Agency account. From there you can traverse our API in GraphQL to the data of your connected schools.

## Viewing a Quote

> To fetch all the course information on a quote

```
query {
  studentQuote(studentQuoteId:1) {
    studentQuoteId
    studentQuoteOptions {
      studentQuoteOptionItems {
        __typename
        
        ... on StudentQuoteOptionCourseItem {
          offeringPriceItem {
            durationAmount
            durationType {
              codeName
            }
            startDate
            endDate
            priceAmount
            priceCurrency {
              code
            }
          }
          courseSnapshot {
            offeringCourseCategory {
              codeName
            }
            name
          }
        }
      }
    }
  }
}
```

> You can copy and paste this cURL command to test it out!

```shell
curl 'https://api.edvisor.io/graphql' 
  -H 'Authorization: Bearer <your_edvisor_api_key>' \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":"query {\n  studentQuote(studentQuoteId:<your_quote_id>) {\n    studentQuoteId\n    studentQuoteOptions {\n      studentQuoteOptionItems {\n        __typename\n        \n        ... on StudentQuoteOptionCourseItem {\n          offeringPriceItem {\n            durationAmount\n            durationType {\n              codeName\n            }\n            startDate\n            endDate\n            priceAmount\n            priceCurrency {\n              code\n            }\n          }\n          courseSnapshot {\n            offeringCourseCategory {\n              codeName\n            }\n            name\n          }\n        }\n      }\n    }\n  }\n}","variables":null}' 
  --compressed 
```

> To fetch all the fee information on a quote

```
query {
  studentQuote(studentQuoteId:1) {
    studentQuoteId
    studentQuoteOptions {
      studentQuoteOptionItems {
        __typename
        
        ... on StudentQuoteOptionFeeItem {
          appliedFeePriceItem {
            startDate
            endDate
            priceAmount
            priceCurrency {
              code
            }
          }
          feeSnapshot {
            name
          }
        }
      }
    }
  }
}
```

> You can copy and paste this cURL command to test it out!

```shell
curl 'https://api.edvisor.io/graphql' 
  -H 'Authorization: Bearer <your_edvisor_api_key>' \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":"query {\n  studentQuote(studentQuoteId:<your_quote_id>) {\n    studentQuoteId\n    studentQuoteOptions {\n      studentQuoteOptionItems {\n        __typename\n        \n        ... on StudentQuoteOptionFeeItem {\n          appliedFeePriceItem {\n            startDate\n            endDate\n            priceAmount\n            priceCurrency {\n              code\n            }\n          }\n          feeSnapshot {\n            name\n          }\n        }\n      }\n    }\n  }\n}","variables":null}' 
  --compressed 
```

> To fetch almost everything on a quote

```graphql
query ($studentQuoteId: Int!) {
  studentQuote(studentQuoteId: $studentQuoteId) {
    studentQuoteId
    # optional: add any student quote field
    studentQuoteStatus {
      studentQuoteStatusId
      codeName
    }
    agency {
      agencyId
      # optional: add any agency field
    }
    student {
      studentId
      # optional: add any student field
    }
    agentUser {
      userId
      # optional: add any user field 
    }
    language {
      code
    }
    filterStudentCurrentCountry {
      countryId
      code
    }
    filterEligibleStudentNationalityCountry {
      countryId
      code
    }
    studentQuoteOptions {
      name
      isAccepted
      totalPrice
      priceCurrencyId
      notes
      priceCurrency {
        currencyId
        code
        symbol
      }
      studentQuoteOptionItems {
        ... on StudentQuoteOptionCourseItem {
          studentQuoteOptionItemId
          studentQuoteOptionItemTypeId
          studentQuoteOptionItemType {
            studentQuoteOptionItemTypeId
            codeName
          }
          isAgencyItem
          offeringPriceItem {
            offeringId
            durationAmount
            durationTypeId
            startDate
            endDate
          }
          courseSnapshot {
            offeringId
            offeringCourseCategoryId
            name
          }
        }
        ... on StudentQuoteOptionAccommodationItem {
          studentQuoteOptionItemId
          studentQuoteOptionItemTypeId
          studentQuoteOptionItemType {
            studentQuoteOptionItemTypeId
            codeName
          }
          isAgencyItem
          offeringPriceItem {
            offeringId
            durationAmount
            durationTypeId
            startDate
            endDate
          }
          accommodationSnapshot {
            offeringId
            offeringAccommodationCategoryId
            bathroomTypeId
            roomTypeId
            mealTypeId
            name
          }
        }
        ... on StudentQuoteOptionServiceItem {
          studentQuoteOptionItemId
          studentQuoteOptionItemType {
            codeName
          }
          isAgencyItem
          offeringPriceItem {
            offeringId
            durationAmount
            durationTypeId
            startDate
            endDate
          }
          serviceSnapshot {
            offeringId
            serviceQuantity
            name
          }
        }
        ... on StudentQuoteOptionFeeItem {
          studentQuoteOptionItemId
          studentQuoteOptionItemType {
            codeName
          }
          isAgencyItem
          appliedFeePriceItem {
            feeId
            appliedToOfferingId
            durationAmount
            durationTypeId
            startDate
            endDate
          }
          feeSnapshot {
            feeId
            name
          }
        }
        ... on StudentQuoteOptionDiscountItem {
          studentQuoteOptionItemId
          studentQuoteOptionItemType {
            codeName
          }
          isAgencyItem
          promotionSnapshot {
            promotionId
            name
          }
          appliedDiscountPriceItem {
            ... on AppliedAmountOffDiscountPriceItem {
              promotionId
              promotion {
                name
              }
              priceAmount
              priceCurrency {
                currencyId
                code
                symbol
              }
              appliedToOffering {
                offeringId
                # optional: add any field on an offering
              }
              appliedToFee {
                feeId
                # optional: add any fee field
              }
              endDate
            }
            ... on AppliedDurationExtensionDiscountPriceItem {
              promotionId
			  priceAmount
              priceCurrency {
                currencyId
                code
                symbol
              }
              endDate
              extensionDurationType {
                durationTypeId
                codeName
              }
              extensionDurationAmount
              promotion {
                name
              }
              appliedToOffering {
                offeringId
                # optional: add any offering field
              }
            }
            ... on AppliedCustomDiscountPriceItem {
              promotionId
              promotion {
                name
              }
              appliedToOffering {
                offeringId
                # optional: add any offering field
              }
              endDate
              discountDescription
            }
            # ... on moreDiscountPriceItem types will be supported in the future
          }
        }
      }
      studentQuoteOptionFiles {
        fileId
        uploaderUserId
        mimeType
        fileExtension
        name
        path
        url
      }
    }
  }
}
```

As an agency customer, you can retrieve data on any of the quotes that you have previously created. Please see the example on the right.

The example to the right shows only how to fetch information for the courses and fees attached to your quote. There are many more objects that you can fetch in addition to course information. Please see the documentation on the following objects for a full description of what data is available to you:

* <a href='http://docs.edvisor.io/schema/studentquoteoptioncourseitem.doc.html'>StudentQuoteOptionCourseItem</a>
* <a href='http://docs.edvisor.io/schema/studentquoteoptionaccommodationitem.doc.html'>StudentQuoteOptionAccommodationItem</a>
* <a href='http://docs.edvisor.io/schema/studentquoteoptionserviceitem.doc.html'>StudentQuoteOptionServiceItem</a>
* <a href='http://docs.edvisor.io/schema/studentquoteoptionfeeitem.doc.html'>StudentQuoteOptionFeeItem</a>
* <a href='http://docs.edvisor.io/schema/studentquoteoptiondiscountitem.doc.html'>StudentQuoteOptionDiscountItem</a>

## Viewing a StudentEnrollment

> To fetch all the salient details of a student enrollment

```
query {
  studentEnrollment(studentEnrollmentId: 1) {
    studentEnrollmentId
    agencyOwnerId
    sent
    studentEnrollmentOfferingItems {
      ... on StudentEnrollmentOfferingCourseItem {
        offeringTypeId
        courseSnapshot {
          offeringCourseCategory {
            offeringCourseCategoryId
            codeName
          }
          name
        }
      }
      ... on StudentEnrollmentOfferingCourseItem {
        accommodationSnapshot {
          bathroomType {
            bathroomTypeId
            codeName
          }
          offeringId
          offeringAccommodationCategoryId
          name
        }
      }
      ... on StudentEnrollmentOfferingServiceItem {
        offeringTypeId
        serviceSnapshot {
          offeringId
        }
      }
    }
  }
}
```

> You can copy and paste this cURL command to test it out!

```shell
curl 'https://api.edvisor.io/graphql' 
  -H 'Authorization: Bearer <your_edvisor_api_key>' \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":"query {\n  studentQuote(studentQuoteId: 42157) {\n    studentQuoteId\n\n    studentQuoteOptions {\n      studentQuoteOptionId\n      \n      studentQuoteOptionItems {\n        ... on StudentQuoteOptionCourseItem {\n          studentQuoteOptionId\n        }\n      }\n    }\n  }\n  \n  studentEnrollment(studentEnrollmentId: 11722) {\n    studentEnrollmentId\n    studentEnrollmentStatusId\n    studentEnrollmentDecisionStatusId\n    schoolId\n    studentId\n    invoiceId\n    quoteId\n    languageId\n    obscuredId\n    studentSignature\n    signedDate\n    specialInstructions\n    created\n    modified\n    agencyOwnerId\n    schoolOwnerId\n    sent\n    isDeletedByAgency\n    isDeletedBySchool\n    studentEnrollmentOfferingItems {\n      ... on StudentEnrollmentOfferingAccommodationItem {\n        accommodationSnapshot {\n          offeringId\n          offeringAccommodationCategoryId\n          name\n        }\n      }\n      ... on StudentEnrollmentOfferingCourseItem {\n        offeringTypeId\n        studentEnrollmentOfferingItemId\n        courseSnapshot {\n          offeringId\n          offeringCourseCategoryId\n          name\n        }\n      }\n    }\n  }\n}","variables":null}'
  --compressed 
```

As an agency customer, you can retrieve data on any of the student enrollments that have been created through Edvisor. Please
see the example on the right. Student enrollments are variously called "applications" or "bookings" in the industry.

The example to the right shows only how to fetch information for the courses, accommodations, and services attached to the student enrollment. For more details as to which fields are available on which objects, consult the schema documentation. 

* <a href='http://docs.edvisor.io/schema/studentenrollmentofferingcourseitem.doc.html'>StudentEnrollmentOfferingCourseItem</a>
* <a href='http://docs.edvisor.io/schema/studentenrollmentofferingaccommodationitem.doc.html'>StudentEnrollmentOfferingAccommodationItem</a>
* <a href='http://docs.edvisor.io/schema/studentenrollmentofferingserviceitem.doc.html'>StudentEnrollmentOfferingServiceItem</a>

## Viewing an Invoice

> Fetch invoices by ID or based on a filtered search

```
query {
	invoice(invoiceId: 1) {
	  invoiceId
	  externalId
	  invoiceTypeId
	  ownerId
	  studentId
	  studentNationalityId
	  studentMinAge
	  studentMaxAge
	  partnershipId
	  invoiceStatusId
	  invoicePaymentStatusId
	  isOverdue
	  notes
	  viewed
	  issued
	  expires
	  paymentCurrencyId
	  depositCurrencyId
	  depositAmount
	  depositIsPercentage
	  depositDateDue
	  depositIsRequired
	  languageId
	  obscuredId
	  createdByUserId
	  lastUpdatedByUserId
	  created
	  modified
    rawItems {
      invoiceItemId
      invoiceId
      invoiceItemTypeId
      offeringId
      promotionId
      snapshotPromotionTypeId
      snapshotExtensionDurationTypeId
      snapshotFreeDurationAmount
      snapshotBonusDurationAmount
      snapshotPromotionApplicableTypeId
      snapshotPromotionSecondaryTypeId
      snapshotPromotionPercentage
      feeId
      isAgencyItem
      isGlobalFee
      position
      school
      program
      durationAmount
      durationTypeId
      notes
      startDate
      endDate
      googlePlaceId
      currencyId
      amount
      isAmountPerDuration
      serviceFeeMultiplier
      partnershipId
      partnershipOfferingId
      created
      modified
    }
    invoiceFiles {
      fileId
      uploaderUserId
      mimeType
      fileExtension
      name
      path
      url
      created
      modified
    }
    invoiceType {
      invoiceTypeId
      codeName
    }
    paymentCurrency {
      currencyId
      code
      symbol
      oneUsd
      enabled
      created
      modified
    }
	} 
}
```

> You can copy and paste this cURL command to test it out!

```shell
curl 'https://api.edvisor.io/graphql' 
  -H 'Authorization: Bearer <your_edvisor_api_key>' \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":"query {\n\tinvoice(invoiceId: 24157) {\n\t  invoiceId\n\t  externalId\n\t  invoiceTypeId\n\t  ownerId\n\t  studentId\n\t  studentNationalityId\n\t  studentMinAge\n\t  studentMaxAge\n\t  partnershipId\n\t  invoiceStatusId\n\t  invoicePaymentStatusId\n\t  isOverdue\n\t  notes\n\t  viewed\n\t  issued\n\t  expires\n\t  paymentCurrencyId\n\t  depositCurrencyId\n\t  depositAmount\n\t  depositIsPercentage\n\t  depositDateDue\n\t  depositIsRequired\n\t  languageId\n\t  obscuredId\n\t  createdByUserId\n\t  lastUpdatedByUserId\n\t  created\n\t  modified\n    rawItems {\n      invoiceItemId\n      invoiceId\n      invoiceItemTypeId\n      offeringId\n      promotionId\n      snapshotPromotionTypeId\n      snapshotExtensionDurationTypeId\n      snapshotFreeDurationAmount\n      snapshotBonusDurationAmount\n      snapshotPromotionApplicableTypeId\n      snapshotPromotionSecondaryTypeId\n      snapshotPromotionPercentage\n      feeId\n      isAgencyItem\n      isGlobalFee\n      position\n      school\n      program\n      durationAmount\n      durationTypeId\n      notes\n      startDate\n      endDate\n      googlePlaceId\n      currencyId\n      amount\n      isAmountPerDuration\n      serviceFeeMultiplier\n      partnershipId\n      partnershipOfferingId\n      created\n      modified\n    }\n    invoiceFiles {\n      fileId\n      uploaderUserId\n      mimeType\n      fileExtension\n      name\n      path\n      url\n      created\n      modified\n    }\n    invoiceType {\n      invoiceTypeId\n      codeName\n    }\n    paymentCurrency {\n      currencyId\n      code\n      symbol\n      oneUsd\n      enabled\n      created\n      modified\n    }\n\t} \n}","variables":null}'
  --compressed 
```

As an agency customer, you can retrieve data on any of the invoices that have been created through Edvisor. Please
see the example on the right. You can structure the query to get additional info on the invoice, such as any files
associated with it, any invoice items (referred to as raw items), the invoice type, and the payment currency.

There are two different endpoints to query invoices, either by Invoice Id or using a filtered search. 

The following parameters are available for the filtered search:
<ul>
  <li>agencyIds: number []</li>
  <li>invoiceIds: number []</li>
  <li>externalIds: number []</li>
  <li>created: DateOnlyRangeInput</li>
</ul>

This query also takes a pagination input, which is an object with an offset and a limit field.

The example on the right shows a filtered search using agencyIds and the created parameter.

The created parameter can take the following filters:
<ul>
  <li>gte: greater than or equal</li>
  <li>gt: greater than</li>
  <li>lte: less than or equal</li>
  <li>lt: less than</li>
  <li>eq: equal</li>
</ul>

```
query {
  invoiceList(
  pagination: { limit: 100, offset:0 }, 
  filter: {
    agencyIds: [1, 2],
    created: {
      gt: "2018-01-31"
    }
  }) {
    count
    data {
      ...
    } 
  }
    
}
```

For more details as to which fields are available on which objects, consult the schema documentation. 

* <a href='http://docs.edvisor.io/schema/invoice.doc.html'>Invoice</a>
* <a href='http://docs.edvisor.io/schema/invoiceitem.doc.html'>InvoiceItem</a>
* <a href='http://docs.edvisor.io/schema/invoicepayment.doc.html'>InvoicePayment</a>


## Agency Company Currency Exchange Rates

> Easily query your Agency Company Exchange Rates and update them as needed

This is an example query object with details on the `fromCurrency` object

```
  agencyCompanyCurrencyRates {
    agencyCompanyId
    fromCurrencyId
    toCurrencyId
    rate
    created
    modified
    fromCurrency {
      currencyId
      code
      symbol
      oneUsd
      enabled
      created
      modified
    }
  }
```

> You can copy and paste the following cURL command to test it out

```shell
curl 'https://api.edvisor.io/graphql' 
  -H 'Authorization: Bearer <your_edvisor_api_key>' \
  -H 'Content-Type: application/json' \
  -data-binary '{"query":"query {\n  agencyCompanyCurrencyRates {\n    agencyCompanyId\n    fromCurrencyId\n    toCurrencyId\n    rate\n    created\n    modified\n    fromCurrency {\n      currencyId\n      code\n      symbol\n      oneUsd\n      enabled\n      created\n      modified\n    }\n  }\n}","variables":{"input":{"fromCurrencyId":8,"toCurrencyId":157,"rate":0.2}},"operationName":null}' 
  --compressed
```

As an agency customer you can make a query to receive the exchange rates set by your agency. 
The query takes no parameters, and returns all the existing exchange rates for your Agency.

For more details as to which fields are available on which objects, consult the schema documentation. 

* <a href='http://docs.edvisor.io/schema/agencycompanycurrencyrate.doc.html'>AgencyCompanyCurrencyRate</a>

Using the currency API you can also update the exchange rates for your Agency Company. The endpoint takes an array of
objects, so you can update multiple currencies with one simple request. 

Examples of how to use the `upsertAgencyCompanyCurrencyRates` endpoint and its expected input are on the right.

> The input to update the currencies is as follows:

```
{
    "fromCurrencyId": 8,
    "toCurrencyId": 157,
    "rate": 0.2
  }
```

> Alternatively, if you do not know the IDs for each currency, you can use the currency code (all caps). For example:

```
{
    "fromCurrencyCode": "CAD",
    "toCurrencyCode": "USD",
    "rate": 0.75
}
```

> You can use either the currency code or the ID. Here is an example of a cURL request to update two currencies:

```shell
curl 'https://api.edvisor.io/graphql' 
  -H 'Authorization: Bearer <your_edvisor_api_key>' \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":"mutation updateCurrencyRate($input: [AgencyCompanyCurrencyRateInput]){\n  upsertAgencyCompanyCurrencyRates(input: $input) {\n    agencyCompanyId\n    fromCurrencyId\n    toCurrencyId\n    rate\n    created\n    modified\n  }\n}","variables":{"input":[{"fromCurrencyId":8,"toCurrencyCode":"XPF","rate":69.8},{"fromCurrencyId":8,"toCurrencyId":154,"rate":42.69},{"fromCurrencyId":8,"toCurrencyId":155,"rate":19.69}]},"operationName":"updateCurrencyRate"}' 
  --compressed
```
