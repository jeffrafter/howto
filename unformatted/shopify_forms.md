<!-- Shopify flash errors (https://rpl.cat/sXQPYzsTMxYaJDwge1r92U7lYX32aC5PWgWnUtosPcM) -->

    <div class="row s-none">
      <div class="box errors">
        <h2>
          There was 1 error with this customer:
        </h2>
        <ul>
          <li>Customer must have a name or email address</li>
        </ul>
      </div>
    </div>

<!-- Shopify overall layout -->

    <div class="section overview">
      <div class="layout-content">
        <!-- Sections go here -->
      </div>
    </div>
    

<!-- Shopify left description (note, this goes with the overall layout) https://rpl.cat/ZlHb_vgYHwUf8nZBvVB3zPOnAsq338qLFk2a5gNeN-0 -->

    <div class="layout-content">
      <div class="layout-content__sidebar layout-content__first">
        <div class="section-summary">
          <h1>Customer overview</h1>
          <p>Some basic information about this customer.</p>
        </div>
      </div>

      <div class="layout-content__main">
        <div class="next-card">
          <div class="next-card__section">
            <div class="next-grid next-grid--no-outside-padding next-grid--inner-grid">
              <div class="next-grid__cell">
                <div class="next-input-wrapper"><label class="next-label" for="customer_first_name">First Name</label><input bind="firstName" class="next-input" id="customer_first_name" name="customer[first_name]" size="30" type="text" value=""></div>
              </div>
              <div class="next-grid__cell">
                <div class="next-input-wrapper"><label class="next-label" for="customer_last_name">Last Name</label><input bind="lastName" class="next-input" id="customer_last_name" name="customer[last_name]" size="30" type="text" value=""></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

<!-- Shopify right side panels (use this instead of the overall layout if you want it)-->

    <div class="next-grid next-grid--outer-grid">
      <div class="next-grid__cell">
        <div class="next-card">
        </div>
      </div>
    </div>            
      
    <div class="next-grid__cell next-grid__cell--third">
      <div class="next-card next-card--aside visibility">
        <section class="next-card__section">
          <div class="next-grid next-grid--no-padding next-grid--vertically-centered">
            <div class="next-grid__cell">
              <h2 class="next-heading next-heading--no-margin">Visibility</h2>
            </div>
          </div>
        </section>
       <section class="next-card__section" style="padding-top: 15px; padding-bottom: 15px;">
          <div class="next-grid next-grid--compact next-grid--no-outside-padding next-grid--vertically-centered">
            <div class="next-grid__cell">
            </div>
          </div>
        </section>  
      </div>
    </div>      
     

<!-- Shopify input -->

    <div class="next-input-wrapper">
      <label class="next-label" for="customer_first_name">First Name</label>
      <input bind="firstName" class="next-input" id="customer_first_name" name="customer[first_name]" size="30" type="text" value="">
    </div>


<!-- Shopify checkbox -->

    <div class="next-input-wrapper">
      <label class="next-label next-label--switch" for="customer_accepts_marketing">Customer accepts marketing</label>
      <input name="customer[accepts_marketing]" type="hidden" value="0">
      <input class="next-checkbox" id="customer_accepts_marketing" name="customer[accepts_marketing]" type="checkbox" value="1">
      <span class="next-checkbox--styled"></span>
    </div>


<!-- Shopify select -->

    <div class="next-input-wrapper">
      <label class="next-label" for="customer_addresses_attributes_0_country">Country</label>
      <div class="next-select__wrapper next-input--has-content">
      <select class="next-select js-country-select" id="customer_addresses_attributes_0_country" name="customer[addresses_attributes][0][country]">
        <option value="">--</option>
      </select>
    </div>


<!-- Shopify expanding textarea -->

    <div class="next-input-wrapper">
      <label class="next-label next-label--is-focused" for="customer_note">Note</label>
      <textarea class="next-textarea next-textarea--expanding" expanding="true" id="customer_note" name="customer[note]" placeholder="Add a note to this customer…" style="height: 112px;"></textarea>
      <!-- Auto expanding textarea pieces here -->
    </div>


<!-- Shopify footer -->

    <div class="section section--no-border">
      <div class="next-grid next-grid--right-aligned">
        <div class="next-grid__cell next-grid__cell--no-flex">
          <a class="btn" href="/admin/customers">Cancel</a>
          <button class="btn js-btn-primary js-btn-loadable has-loading disabled" name="button" type="submit" disabled="disabled">Save customer</button>
        </div>
      </div>
    </div>


<!-- Shopify error -->

    <div class="next-input-wrapper next-input-wrapper--is-error">
      <!-- ... -->
    </div>


As the form changes the save button becomes active. If the form returns to the original state the button deactivates.

