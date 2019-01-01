---
layout: page
title: Search
---

<script>
  function SearchThisSite()
  {
    var url="https://www.bing.com/search?q=site%3A{{ site.url }}+" + document.getElementById("url").value;
    location.href=url;
    return false;
  }
</script>

<form onsubmit="return SearchThisSite();">
  <div class="search-form-container">
    <div class="search-input-container">
      <input class="search-input" type="text" name="url" id="url" />
    </div>
    <div class="search-submit-container">
      <input class="search-submit" type="submit" value="Go" />
    </div>
  </div>
</form>