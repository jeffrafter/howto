# JavaScript Popups

Creating modal popups in JavaScript is always complex. Here is a quick pattern for doing it.

## Stylesheet

Start with the basic stylesheet `popup.css`:

    div.popup-background {
      z-index: 99999;
      display: none;
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: #333333;
    }

    div.popup {
      z-index: 999999;
      display: none;
      position: absolute;
      top: 4em;
      width: 80%;
      max-width: 486px;
      left: 50%;
      margin-left: -243px;
      background-color: white;
      -moz-box-shadow: 0 8px 46px rgba(0, 0, 0, .5);
      -o-box-shadow: 0 8px 46px rgba(0, 0, 0, .5);
      -webkit-box-shadow: 0 8px 46px rgba(0, 0, 0, .5);
      box-shadow: 0 8px 46px rgba(0, 0, 0, .5);
    }

    div.popup p {
      padding: 1.2em;
    }

    div.popup form {
      margin: 0 auto;
      padding: 1.2em;
      margin-bottom: 1em;
      width: 100%;
    }

    div.popup a.close {
      position: absolute;
      top: 0.2em;
      right: 0.5em;
      font-size: 2em;
      font-weight: 300;
    }

    div.popup a.close:hover {
      border-color: #d2d2d2;
      color: #d2d2d2;
    }

    div.popup h2 {
      font-weight: 300;
      font-size: 1.6em;
      width: 100%;
      margin: 1em 0 0 0;
      padding: 0 0.8em;
      text-align: left;
    }


## Popup code

Include the following `popup.js` in your assets:

    var Popup = function($template) {
      var self = this;

      this.showing = false;
      this.$template = $template;
      this.$background = $("<div class='popup-background'></div>")
      this.$background.click(function(e) {
        e.preventDefault();
        e.stopPropagation();
        self.hide();
      });
      $('body').append(this.$background);
    }

    Popup.prototype.show = function(options) {
      if (this.showing) return false;
      options = options || {};

      var self = this;
      this.$popup = this.$template.clone();
      this.$popup.find('a.close').click(function(e) {
        self.hide();
        e.preventDefault();
        e.stopPropagation();
        return false;
      });

      // Escape closes the popup
      $('body').bind('keyup.popup', function(e) {
        if (e.which === 27) {
          e.preventDefault();
          e.stopPropagation();
          self.hide();
        }
      });

      // No scroll mobile
      $('body').bind('touchmove.popup', function(e) {
        e.preventDefault();
        e.stopPropagation();
      });

      if (options.before) {
        options.before.apply(this.$popup, [])
      }

      $('body').addClass('popup');
      $('body').append(this.$popup);
      this.$popup.css('top', $(window).scrollTop() + 50);
      this.$popup.fadeIn(300);
      this.$background.css({'opacity': '0.8'});
      this.$background.fadeIn(300);
      this.showing = true;
    }

    Popup.prototype.hide = function() {
      if (!this.showing) return false;

      var self = this;

      this.$background.fadeOut(300);
      this.$popup.fadeOut(300, function() {
        self.$popup.remove();
        self.showing = false;
      });

      $('body').removeClass('popup');
      $('body').unbind('touchmove.popup'); // no scroll mobile unbind
      $('body').unbind('keyup.popup'); // escape unbind
    }



## Usage

In order to use this you need a template for your popup in your HTML. You can put this just above the footer (or in a page specific template):

    <div class="popup sample">
      <h2>Title</h2>
      <div>
        <p>Something about something</p>
        <form action="#" onsubmit="false">
          <button>Hide</button>
        </form>
      </div>
      <a href="javascript:void(0)" class="close">&times;</a>
    </div>

You'll also need a temporary lightbox background element. I include this in the application layout directly:

    <div id="lightbox-background"></div>

Then you need a small amount of JavaScript to register the popup and show it:

    <script type="text/javascript">
      $(function() {
        var samplePopup = new Popup($('.sample'));

        samplePopup.show({
          before: function() {
            var $popup = this;
            $popup.find('button').click(function() {
              samplePopup.hide();
            });
          }
        });
      });
    </script>
