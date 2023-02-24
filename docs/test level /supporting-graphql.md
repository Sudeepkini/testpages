# Supporting GraphQL APIs in your modules
To overcome the redundancy of Rest APIs, Graphql Migration is taking place from Backend Teams. Currently, Groceries Squads are using Graphql APIs in their module from the FE side.
You can find the below document useful in case you want to integrate new APIs in your module which is based on Graphql
[GraphQL B2CAndroid](https://docs.google.com/document/d/1mnLpUnM7yi6yA2eDMkqeifk6p_SDZnsEJkcFRlDihls/edit?usp=sharing)

# The problem with Rest API
Most of the APIs currently used in the b2c project are GET,POST in the application but divided according to the module requirement. Retrofit is used in getting Network API calls in below manner in each module.



    @Provides
    @Reusable
    internal fun provideGraphQLCampaignApi(serviceProvider: PandoraServiceProvider): GraphQLGroceryProductServiceApi {
        return serviceProvider.createService() {
            setConverterFactory(ApolloConverterFactory())
        }
    }

    @Provides
    @Reusable
    internal fun providesBannersRetrofitApi(discoServiceProvider: DiscoServiceProvider): DjiniServiceApi {
        val gson = GsonBuilder().setDateFormat(DATE_FORMAT_DJINI).create()
        val converterFactory = GsonConverterFactory.create(gson)
        return discoServiceProvider.createService(converterFactory = converterFactory)
    }
**Disco** And **Pandora** are the main backend base end points for Front End(FE). If you want to provide any other base end points then please check implementation of the ServiceProvider like Pandora Service Provider and implement same in _api-module_ of your core module.
