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

`entrySaleSubmit()` function calls a ajax 
