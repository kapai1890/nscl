# WordPress Settings
Workaround for functions add_settings_*(), register_setting().

# Requirements
* PHP 5.6
* WordPress 4.9

# Purpose
It's for doing this:
```php
SettingsRegistry::registerSettings(
    'general',
    [
        'todebug' => [
            0 => __('Todebug', 'todebug'),
            'todebug_custom_file' => [
                'title'   => __('Custom File', 'todebug'),
                'type'    => 'string',
                'default' => ''
            ]
        ]
    ]
);
```
... Instead of:
```php
class GeneralSettings
{
    public function __construct()
    {
        $this->addActions();
    }

    protected function addActions()
    {
        add_action('admin_init', [$this, 'registerGroups']);
        add_action('admin_init', [$this, 'registerFields']);
    }

    public function registerGroups()
    {
        $echoNothing = function () {};

        add_settings_section('todebug', __('Todebug', 'todebug'), $echoNothing, 'general');
    }

    public function registerFields()
    {
        add_settings_field('todebug_custom_file', __('Custom File', 'todebug'),
            [$this, 'renderCustomFile'], 'general', 'todebug');

        register_setting('general', 'todebug_custom_file', ['type' => 'string', 'default' => '']);
    }

    public function renderCustomFile()
    {
        $logFile = get_option('todebug_custom_file', '');

        echo '<input id="todebug_custom_file" name="todebug_custom_file" type="text" value="' . esc_attr($logFile) . '" class="large-text" />';
    }
}
```

# Arguments List
```php
SettingsRegistry::registerSettings(
    "%page name%", // general|writing|reading|discussion|media|permalink (but not "privacy")
    [
        "%section name #1%" => [
            0 => "%section title%",

            "%field name #1%" => [
                // All arguments are optional

                // Arguments for register_setting()
                "type"              => "string", // boolean|number|integer|string
                "description"       => "", // Also description on settings page.
                                           // Plain text or HTML (already escaped)
                "sanitize_callback" => SettingsField::sanitizeValue(),
                "show_in_rest"      => false,
                "default"           => "", // Can be array in MulticheckField

                // Additional arguments for SettingsField and SettingsRegistry
                "title"             => "Title text on the left side of the page",
                "input_type"        => "text", // text|number|checkbox|select|radio|multicheck
                "class"             => "", // CSS classes

                // TextField
                "placeholder"       => "",
                "size"              => "regular", // tiny|small|regular|large|""

                // NumberField
                "size"              => "small", // tiny|small|regular|large|""
                "label"             => "", // The label next after the field
                "min"               => "", // Number or ""
                "max"               => "", // Number or ""
                "step"              => "", // Number or ""

                // CheckboxField
                "label"             => "", // The label next after the field.
                                           // Plain text or HTML (escaped)

                // SelectField
                "options"           => [%value% => %label%],

                // RadioField
                "options"           => [%value% => %label%],

                // MulticheckField
                "values"            => [%value% => %label%],
                "separator"         => "," // Values separator in wp_options
            ],

            "%field name #2%" => ...
        ],

        "%section name #2%" => ...
    ]
);
```
