# ui-animator
AngularJS lib for animation


<pre><code>

'use strict';


angular.module('ui.animator', ['ui.animator.spinner']);

/*
* ui.animator.spinner
*
* This module exposes different spinner components regarding the spin.js lib.
*
* */
angular.module('ui.animator.spinner',['ui.animator.spinner.spinButtonOn']);


/*
 * ui.animator.spinner.spinButtonOn
 *
 * This module exposes a spinner which works on button.
 *
 * */
angular.module('ui.animator.spinner.spinButtonOn',[])
    /*
    *
    * This directive is used with the html <button> tag and works especially with bootstrap and glyphicon icons.
    *
    * The idea is when the button is clicked and an action is pending, we replace the target element (ex:icon) inside the button with a spinner.
    * when the action has finished.
    *
    * options :
    *   {
    *       state: false,                       (boolean) : it drives when the spinner appears and disappears.
    *       targetId: "myDivId"                 (String)  : this is the html element id which be replaced by the spinner.
    *       targetClass: "myClass"              (String)  : this is the html element class which be replaced by the spinner.
    *       options: {}                         (Object)  : this is the overriding options object for the spinner (see spin.js docs)
    *                                                       Not required, if not setted, it will scale automatically the spinner regarding the size of the target.
    *                                                       otherwise the object is like this :
    *                                                       options:{style:'line/circle', size:'s/m', spinnerOptions: {overriding options of spin.js}}
    *   };
    *
    * */
    .directive('spinButtonOn', function() {
        return {
            scope:{
                config:'=spinButtonOn'
            },
            controller: function($scope){
                var defaultColor = '#333333';
                var baseOptions = {width:1,lines:15,scale:0.60,length:0,radius:6,corners:1.0,opacity:0.15,rotate:35,direction:1,speed:1.0,trail:79,color:defaultColor};
                var scOpts = angular.extend(angular.copy(baseOptions), {});
                var slOpts = angular.extend(angular.copy(baseOptions), {width:1,lines:11,scale:0.5,length:6});
                var mcOpts = angular.extend(angular.copy(baseOptions), {width:2,lines:13,scale:1,radius:5});
                var mlOpts = angular.extend(angular.copy(baseOptions), {width:1,lines:13,scale:0.75,length:5});
                $scope.baseOptions = baseOptions;

                if($scope.config.options && !$scope.config.options.spinnerOptions){
                    if($scope.config.options.size=='s'){
                        if($scope.config.options.style=='line'){
                            $scope.config.options.spinnerOptions = slOpts;
                        } else {
                            $scope.config.options.spinnerOptions = scOpts;
                        }
                    } else{
                        if($scope.config.options.style=='line'){
                            $scope.config.options.spinnerOptions = mlOpts;
                        } else {
                            $scope.config.options.spinnerOptions = mcOpts;
                        }
                    }
                } else {
                    $scope.config.options= {spinnerOptions :null};
                }
                $scope.config.uuid = JsTools.uuid.create();
                $scope.container = document.createElement("span");
                $scope.container.setAttribute("id", $scope.config.uuid);
            },
            link : function(scope, element) {
                scope.element = element[0];
                scope.target = scope.config.targetClass ? scope.element.querySelector("."+scope.config.targetClass+"") : scope.element.querySelector("[id='"+scope.config.targetId+"']");
                scope.targetParent = scope.target.parentNode;
                if(!scope.element.querySelector("[id='"+scope.config.uuid+"']")){
                    scope.targetParent.insertBefore(scope.container, scope.target);
                }
                scope.$watch('config.state', function(state){
                    if(state){
                        if(!scope.config.options.spinnerOptions){
                            var opts = JsTools.spinHelper.autoConfig(scope.baseOptions, scope.target);
                            scope.spinner  = new Spinner(opts).spin(scope.container);
                        } else {
                            scope.spinner  = scope.spinner ? scope.spinner.spin(scope.container) : new Spinner(scope.config.options.spinnerOptions).spin(scope.container);
                        }
                        scope.container.setAttribute("style", "display:inline-block;position:relative;line-height:1;height:"+scope.target.offsetHeight+"px;width:"+ scope.target.offsetWidth+"px");
                        scope.target.style.display = 'none';
                    } else {
                        scope.container.style.display = 'none';
                        scope.target.style.display = 'inline-block';
                        scope.spinner ? scope.spinner.stop() : null;
                    }
                });
                element.on('$destroy', function() {
                    scope.spinner.stop();
                    delete scope.spinner;
                });
            }
        };
});

(function(){
    var _JsTools = function(){};
    _JsTools.prototype.uuid = {
        create: function(){
              var s = [];
              var hexDigits = "0123456789abcdef";
              for (var i = 0; i < 36; i++) {s[i] = hexDigits.substr(Math.floor(Math.random() * 0x10), 1);}
              s[14] = "4";
              s[19] = hexDigits.substr((s[19] & 0x3) | 0x8, 1);
              s[8] = s[13] = s[18] = s[23] = "-";
              return s.join("");
        }
    };

    _JsTools.prototype.math = {
        scale : function(o){
            var value = Math.sqrt(o.height*o.width);
            var result = ((o.scaleMax - o.scaleMin) * (value - o.baseMin) / (o.baseMax - o.baseMin)) + o.scaleMin;
            result = Math.round(result * 100) / 100;
            return result ;
        }
    };

    _JsTools.prototype.spinHelper = {
        autoConfig : function(options, elt){
            var baseMin = 11;
            var baseMax = 284;
            var scaleObject = {
                baseMin: baseMin,
                baseMax: baseMax,
                scaleMin: 0.6,
                scaleMax: 5,
                height: elt.offsetHeight ?  elt.offsetHeight : baseMin,
                width: elt.offsetWidth ? elt.offsetWidth : baseMin
            };
            var lengthObject = {
                baseMin: baseMin,
                baseMax: baseMax,
                scaleMin: 0,
                scaleMax: 10,
                height: elt.offsetHeight ?  elt.offsetHeight : baseMin,
                width: elt.offsetWidth ? elt.offsetWidth : baseMin
            };
            var radiusObject = {
                baseMin: baseMin,
                baseMax: baseMax,
                scaleMin: 6,
                scaleMax: 12,
                height: elt.offsetHeight ?  elt.offsetHeight : baseMin,
                width: elt.offsetWidth ? elt.offsetWidth : baseMin
            };
            var scaleValue = JsTools.math.scale(scaleObject);
            var lengthValue = JsTools.math.scale(lengthObject);
            var radiusValue = JsTools.math.scale(radiusObject);
            return angular.extend(angular.copy(options), {scale:scaleValue, length: lengthValue, radius: radiusValue});
        }
    }

    window.JsTools = new _JsTools();
})();

</code></pre>
