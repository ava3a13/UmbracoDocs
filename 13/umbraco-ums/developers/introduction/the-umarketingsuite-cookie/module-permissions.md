# Module Permissions

Starting from version **1.13.0** it is possible to disable the individual modules of uMarketingSuite (Analytics, A/B testing, Personalization) through code based on any criteria you want. For example, you could choose to give visitors control over these settings through a cookiebar on your site. In order to do this you will have to create an implementation of the **uMarketingSuite.Business.Permissions.ModulePermissions.IModulePermissions** interface and override our default implementation which enables everything by default all the time.

This interface defines 3 methods which you will have to implement:

    /// <summary>/// Indicates if A/B testing is allowed for the given request context./// If false, the visitor will not be assigned to any A/B tests and will not/// see any active A/B test content./// </summary>/// <param name="context">Context of the request</param>/// <returns>True if A/B testing is allowed, otherwise false.</returns>bool AbTestingIsAllowed(HttpContextBase context);/// <summary>/// Indicates if Analytics is allowed for the given request context./// If false, the visitor will be treated as the built-in Anonymous visitor/// and all their activity will be assigned to the Anonymous visitor rather than the specific visitor./// No A/B testing or Personalization will be allowed either if this is false regardless of their/// respective IsAllowed() outcomes./// In addition, no cookie will be sent to the visitor when this is set to false./// </summary>/// <param name="context">Context of the request</param>/// <returns>True if Analytics is allowed, otherwise false.</returns>bool AnalyticsIsAllowed(HttpContextBase context);/// <summary>/// Indicates if Personalization testing is allowed for the given request context./// If false, the visitor will not see any personalized content./// </summary>/// <param name="context">Context of the request</param>/// <returns>True if Personalization is allowed, otherwise false.</returns>bool PersonalizationIsAllowed(HttpContextBase context);

Using these methods you can control per visitor whether or not the modules are active. Your implementation will need to be registered with Umbraco using the **RegisterUnique()** method, overriding the default implementation which enables all modules all the time. Make sure your composer runs after the uMarketingSuite composer by using the **[ComposeAfter]** attribute.

For uMarketingSuite 1.x, your composer could look something like this:

    using uMarketingSuite.Business.Permissions.ModulePermissions;using uMarketingSuite.Common.Composing;using Umbraco.Core;using Umbraco.Core.Composing;namespace YourNamespace {    [ComposeAfter(typeof(AttributeBasedComposer))]    public class YourComposer : IUserComposer    {        public void Compose(Composition composition)        {            composition.RegisterUnique<IModulePermissions, YourCustomModulePermissions>();        }    }}

For uMarketingSuite 2.x, the AttributeBasedComposer has been renamed to UMarketingSuiteApplicationComposer, with which it could look something like this:

    using uMarketingSuite.Business.Permissions.ModulePermissions;using uMarketingSuite.Common.Composing;using Umbraco.Core;using Umbraco.Core.Composing;namespace YourNamespace {    [ComposeAfter(typeof(UMarketingSuiteApplicationComposer))]    public class YourComposer : IComposer    {        public void Compose(Composition composition)        {            composition.RegisterUnique<IModulePermissions, YourCustomModulePermissions>();        }    }}

### Important! Tracking a visitors Initial Pageview

If you change the default module permissions to false and the visitor has not given any consent yet the uMarketingSuite does not actively track that visitor until they have given their consent to the Analytics module (module permission AnalyticsIsAllowed set to **true**). If the module permission is set to true it is required to **reload the current page as soon as the visitor has given consent** in order to track the current page visit the visitor has given consent on. If no reload is performed the visitors referrer and/or campaign information will not be tracked!

Calling the "window.location.reload();" method would be the preferred option, as this will preserve any referrers & query strings supplied in the current request, resulting in uMarketingSuite processing the current page visit & visitor correctly. 

An example implementation using Cookiebot can be found [here](/security-privacy/gdpr/how-to-become-gdpr-compliant-using-cookiebot/).