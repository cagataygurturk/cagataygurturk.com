---
layout: post
title:  "ZF2 Controller Specific 404 Not Found Page"
date:  2015-04-17 14:24:00
categories: web
---

As we all know, Zend Framework 2 handles 404 not found event, so whenever you set HttpResponse's status code in a controller with

{% highlight php %}

$response = $this->getResponse();
$response->setStatusCode(404);

{% endhighlight %}

ZF2, by default, intercepts the request and prints the default 404 page. For SEO purposes we should use wisely the status codes. At Instela, when a user types a mispelled word or tries to reach a removed page, we show similar results rather than the default 404 page but also for search engine spiders we give 404 status code at the background. (A mispelling example: <a href="https://tr.instela.com/cagatay-gurtrk--9306406" target="_blank">https://tr.instela.com/cagatay-gurtrk--9306406</a>) <a href="http://webmasters.stackexchange.com/questions/68256/404-code-header-for-search-engines-on-removed-user-content" target="_blank">This is the right way</a> to handle status codes but as I said before ZF2 does not allow us to do it by default.

I asked this question on <a href="http://stackoverflow.com/questions/25736829/zf2-controller-specific-404-error-page">Stackoverflow</a> but I did not receive any satisfying answer. Then I came up with this idea to solve this problem. The idea here is on bootstrap event check in which controller we are (in our case "Title") and if the check is true, iterating over attached event handlers and detach a specific one, in this case "Zend\Mvc\View\RouteNotFoundStrategy".

In Module.php

{% highlight php %}

public function onBootstrap(EventInterface $e) {
        $em = $e->getApplication()->getEventManager();
        $em->attach(MvcEvent::EVENT_ROUTE, array($this, 'detachNotFoundStrategy'), 1);
}

public function detachNotFoundStrategy(MvcEvent $event) {
        $controller = $event->getRouteMatch()->getParam('controller');
        if ($controller != 'Title') {
            return false;
        }
        $app = $event->getApplication();
        $services = $app->getServiceManager();
        $events = $app->getEventManager();
        $sharedEvents = $events->getSharedManager();


        $listener = $services->get('Zend\Mvc\View\RouteNotFoundStrategy');
        $events->detach($listener);
        /** @var CallbackHandler[] $handlers */
        $handlers = $sharedEvents->getListeners('Zend\Stdlib\DispatchableInterface', MvcEvent::EVENT_DISPATCH);
        foreach ($handlers as $handler) {
            $callback = $handler->getCallback();
            if (is_array($callback) && $callback[0] === $listener) {
                $sharedEvents->detach('Zend\Stdlib\DispatchableInterface', $handler);
            }
        }
}

{% endhighlight %}

Easy, right?