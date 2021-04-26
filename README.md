# Laravel nova page settings

Ad hoc solution to add settings configuration to laravel nova.\
The package is currently unstable and support is very slow. For production, you may need to fork

## Installation

You can install the package via composer:

```bash
composer require yaroslawww/nova-page-settings
# optional publish configs
php artisan vendor:publish --provider="ThinkOne\NovaPageSettings\ServiceProvider" --tag="config"
```

## Usage

### Admin part

1. Create settings table

```injectablephp
public function up() {
    Schema::create( 'my_pages_settings', function ( Blueprint $table ) {
        \Thinkone\NovaPageSettings\MigrationHelper::defaultColumns($table);
    } );
}
```

2. Create model

```injectablephp
use Thinkone\NovaPageSettings\PageSetting;

class MyPageSetting extends PageSetting
{
    protected $table = 'my_pages_settings';
}
```

3. Create adapter for settings

```injectablephp
namespace App\Nova\PageSettings\Adapters;

use App\Models\MyPageSetting;
use Thinkone\NovaPageSettings\QueryAdapter\InternalSettingsModel;

class MyPageSettingModel extends InternalSettingsModel
{
    /**
     * @inheritDoc
     */
    public function getDBModel(): string
    {
        return MyPageSetting::class;
    }

    // OPTIONAL, if you need to change default directory folder
    public function getTemplatesPath(): string
    {
        return 'app/Nova/PageSettings/Templates/OtherDirectory';
    }
}
```

4. Create resource

```injectablephp
namespace App\Nova\Resources;

use App\Nova\PageSettings\Adapters\MyPageSettingModel;
use Thinkone\NovaPageSettings\AbstractSettingsResource;

class MyPageSetting extends AbstractSettingsResource
{
    /**
     * The model the resource corresponds to.
     *
     * @var string
     */
    public static $model = MyPageSettingModel::class;
}
```

5. Create templates in folder what you specified in step 3. System will find all templates in folder automatically.

```injectablephp
namespace App\Nova\PageSettings\Templates\OtherDirectory;

use Laravel\Nova\Fields\Image;
use Laravel\Nova\Fields\Text;
use Laravel\Nova\Fields\Textarea;
use Laravel\Nova\Http\Requests\NovaRequest;
use Thinkone\NovaPageSettings\Templates\BaseTemplate;
use Whitecube\NovaFlexibleContent\Flexible;

class MarketingPageSettings extends BaseTemplate
{
    public static function getName(): string
    {
        return 'Marketing Page Settings';
    }

    public function fields(NovaRequest $request)
    {
        return [
            // NOTICE: do not forget use "templateKey" method
            Flexible::make('Slider', $this->templateKey('slider'))
                    ->addLayout('Custom Slide', 'custom_slide', [
                        Image::make('Image', 'image')
                             ->rules([ 'max:' . 1024 * 10 ])
                             ->deletable(false)
                             ->help('~1600x1200px. Already optimised jpg or png. If empty, a random image will be used'),
                        Text::make('Title', 'title'),
                        Textarea::make('Text', 'text'),
                        Text::make('Button Text', 'btn_text'),
                        Text::make('Button Link', 'btn_link'),
                    ])
                    ->limit(4)
                    ->button('Add Slide'),
            // ... other fields
        ];
    }
}
```

5. As a result, you should see something like this:

![](docs/assets/settings-example.gif)

### Frontend part

```injectablephp
 /** @var \Illuminate\Support\Collection $pageSettings */
 $pageSettings =  MyPageSettingModel::page(MarketingPageSettings::getSlug())->get();
 // get array of slides using helper
 $slides = page_setting_value( $pageSettings, 'slider', 'array', [] );
```

## Credits

- [![Think Studio](https://yaroslawww.github.io/images/sponsors/packages/logo-think-studio.png)](https://think.studio/)