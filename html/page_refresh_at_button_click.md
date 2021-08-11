# How to prevent page refresh when a button is clicked?

I coded my html file as following.

```html
<form class="form-inline justify-content-center">
    <div class="form-group mb-2 mx-sm-2">
        <select id="entrySaleSelect" class="custom-select" style="width:auto;">
            <option selected disabled hidden>choose a product</option>
            <option th:each="item : ${productIdList}" th:value="${item}" th:utext="${item}"></option>
        </select>
    </div>
    <button class="btn btn-primary mb-2" onclick="entrySaleSubmit();">Submit</button>
</form>
```

`entrySaleSubmit()` function calls a ajax function and updates text content. That's it.
However when the button is clicked, `entrySaleSubmit()` function is executed as expected but **PAGE REFRESHES**, which is not my intention.

### Fix
Add `type="button"` attribute to `<button>` tag. Unless the buttom is interpreted as `submit` of the form that surrounds it.

```html
<button type="button" class="btn btn-primary mb-2" onclick="entrySaleSubmit();">Submit</button>
``` 
