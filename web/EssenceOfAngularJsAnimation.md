# Which elements?
<table>
    <thead>
        <tr>
            <td>
                <b>Directive</b>
            </td>
            <td>
                <b>Animation Type</b>
            </td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                ngRepeat
            </td>
            <td>
                enter, leave and move
            </td>
        </tr>
        <tr>
            <td>
                ngView
            </td>
            <td>
                enter and leave 
            </td>
        </tr>
        <tr>
            <td>
                ngInclude
            </td>
            <td>
                enter and leave
            </td>
        </tr>
        <tr>
            <td>
                ngSwitch
            </td>
            <td>
                enter and leave
            </td>
        </tr>
        <tr>
            <td>
                ngIf
            </td>
            <td>
                enter and leave
            </td>
        </tr>
        <tr>
            <td>
                ngShow and ngHide
            </td>
            <td>
                show and hide
            </td>
        </tr>
    </tbody>
</table>

# How?
+ CSS Transitions
+ CSS Animations
+ Javascript

# How it works?
When an animation occurs, such as ngView's show: angular will attach a class `.animate-enter` to the element at the beginning, and then attach another class `.animate-enter.animate-enter-active`. Which we need to do is to define these classes in the css file:

```
.animate-enter {
	-webkit-transition: 1s linear all; /* Chrome */
	transition: 1s linear all;
	opacity: 0;
}
 
.animate-enter.animate-enter-active {
	opacity: 1;
}
```

and in the html:

```
<!-- Short Hand -->
<div ng-repeat="item in itens" ng-animate=" 'animate' ">
    ...
</div>
 
<!-- Expanded -->
<div ng-repeat="item in itens" ng-animate="{enter: 'animate-enter', leave: 'animate-leave'}">
```

The short hand is equivalent to the expanded hand, the attribute name `'animate'` is all up to you. You can use different name for different animation.

# CSS animation example via @keyframe
```
.animate-enter { 
    -webkit-animation: enter 600ms cubic-bezier(0.445, 0.050, 0.550, 0.950);
    animation: enter 600ms cubic-bezier(0.445, 0.050, 0.550, 0.950);
    display: block;
    position: relative;
} 

@keyframes enter {
    from {
        opacity: 0;
        height: 0px;
        left: -70px;
    }
    75% {
        left: 15px;
    }
    to {
        opacity: 1;
        height: 30px;
        left: 0px;
    }
}
 
.animate-leave {
    -webkit-animation: leave 600ms cubic-bezier(0.445, 0.050, 0.550, 0.950);
    animation: leave 600ms cubic-bezier(0.445, 0.050, 0.550, 0.950);
    display: block;
    position: relative;
}
@keyframes leave {
    to {
        opacity: 0;
        height: 0px;
        left: -70px;
    }
    25% {
        left: 15px;
    }
    from {
        opacity: 1;
        height: 30px;
        left: 0px;
    }
}
```