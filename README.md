# laravel-Excel-
laravel 下excel导入数据，逻辑：先上传excel文件，然后读取表中数据进行插入数据表
此为百度搜寻后自己修改后的
1、简介
Laravel Excel 在 Laravel 5 中集成 PHPOffice 套件中的 PHPExcel，从而方便我们以优雅的、富有表现力的代码实现Excel/CSV文件的导入和导出。
本文我们将在Laravel中使用Laravel Excel简单实现Excel文件的导入和导出。
2、安装&配置
使用Composer安装依赖
首先在Laravel项目根目录下使用Composer安装依赖：
composer require maatwebsite/excel ~2.0.0
安装后的设置
在config/app.php中注册服务提供者到providers数组：

Maatwebsite\Excel\ExcelServiceProvider::class,
同样在config/app.php中注册门面到aliases数组：
'Excel' => Maatwebsite\Excel\Facades\Excel::class,
如果想要对Laravel Excel进行更多的自定义配置，执行如下Artisan命令：
php artisan vendor:publish
执行成功后会在config目录下生成一个配置文件excel.php。

3、导出Excel文件
为了演示Laravel Excel相关功能，我们为本测试创建一个干净的控制器ExcelController.php：
php artisan make:controller ExcelController --plain
然后在routes.php中定义相关路由：
Route::get('excel/export','ExcelController@export');
Route::get('excel/import','ExcelController@import');
接下来我们先在ExcelController.php中定义export方法实现导出功能：
<?php
namespace App\Http\Controllers;
use Illuminate\Http\Request;
use App\Http\Requests;
use App\Http\Controllers\Controller;
use Excel;
class ExcelController extends Controller
{
    //Excel文件导出功能 By Laravel学院
    public function export(){
        $cellData = [
            ['学号','姓名','成绩'],
            ['10001','AAAAA','99'],
            ['10002','BBBBB','92'],
            ['10003','CCCCC','95'],
            ['10004','DDDDD','89'],
            ['10005','EEEEE','96'],
        ];
        Excel::create('学生成绩',function($excel) use ($cellData){
            $excel->sheet('score', function($sheet) use ($cellData){
                $sheet->rows($cellData);
            });
        })->export('xls');
    }
}
我们在浏览器中访问http://laravel.app:8000/excel/export，会导出一个名为学生成绩.xls的Excel文件：
使用Laravel Excel导出文件
如果你要导出csv或者xlsx文件，只需将export方法中的参数改成csv或xlsx即可。
如果还要将该Excel文件保存到服务器上，可以使用store方法：
Excel::create('学生成绩',function($excel) use ($cellData){
     $excel->sheet('score', function($sheet) use ($cellData){
         $sheet->rows($cellData);
     });
})->store('xls')->export('xls');
文件默认保存到storage/exports目录下，如果出现文件名中文乱码，将上述代码文件名做如下修改即可：
iconv('UTF-8', 'GBK', '学生成绩')
4、导入Excel文件
我们将刚才保存到服务器上的Excel文件导入进来，导入很简单，使用Excel门面上的load方法即可：
//Excel文件导入功能 By Laravel学院
public function import(){
    public function import()
    {
        ini_set('max_execution_time', 300);
        ini_set('max_input_time ', 300);
        $file = Input::file('myfile');
        if($file){
//          $realPath = $file
//          $path = $file -> move(app_path().'/storage/uploads');
            $realPath = $file->getRealPath();
            $entension =  $file -> getClientOriginalExtension(); //上传文件的后缀.
            $tabl_name = date('YmdHis').mt_rand(100,999);
            $newName = $tabl_name.'.'.'xls';//$entension;
            $path = $file->move(base_path().'/uploads',$newName);
            $cretae_path = base_path().'/uploads/'.$newName;

            //dd($cretae_path);
            //dd($file);
            //读取excel的数据
            Excel::load($cretae_path, function($reader) use($tabl_name){
                //$data = $reader->all();
                //获取excel的第几张表
                $reader = $reader->getSheet(0);
                //获取表中的数据
                $data = $reader->toArray();
                ExcelTmp::truncate();
                $result = $this->insert_data($data); //这里是做的插入数据库表的方法 可以自行填写
            });
        }
    }
      
}


上述步骤完成后需要修改php.ini 完成大文件上传，或者在程序里进行设置，主要参数如下
第一步：修改在php5下POST文件大小的限制

1.编修php.ini

找到：max_execution_time = 30 ，这个是每个脚本运行的最长时间，单位秒，修改为：
max_execution_time = 150

找到：max_input_time = 60，这是每个脚本可以消耗的时间，单位也是秒，修改为：
max_input_time = 300

找到：memory_limit = 128M，这个是脚本运行最大消耗的内存，根据你的需求更改数值，这里修改为：
memory_limit = 256M

找到：post_max_size = 8M，表单提交最大数据为 8M，此项不是限制上传单个文件的大小,而是针对整个表单的提交数据进行限制的。限制范围包括表单提交的所有内容.例如:发表贴子时,贴子标题,内容,附件等…这里修改为：
post_max_size = 20M

找到：upload_max_filesize = 2M ，上载文件的最大许可大小 ，修改为：
upload_max_filesize = 10M

第二步： Apache环境中的档案上传大小控制
修改位于Apahce目录下的httpd.conf
添加下面内容
LimitRequestBody 10485760    
即10M=10*1024*1024，有的文章中提到应改为 600000000
重新启动apache，就可以在设置里看到你要的大小

Linux 环境下的修改方法 ================================================================
修改etc/php.ini
找到 File Uploadsh区域修改以下几个参数： file_uploads = on ;是否允许通过HTTP上传文件的开关。默认为ON即是开 upload_tmp_dir ;文件上传至服务器上存储临时文件的地方，如果没指定就会用系统默认的临时文件夹(moodle可以不改)
upload_max_filesize = 8m ;允许上传文件大小的最大值.
找到 Data Handling区域，修改 post_max_size = 8m ;指通过表单POST给PHP的所能接收的最大值，包括表单里的所有值。默认为8M
设上述四个参数后，上传小于8M的文件一般不成问题。但如果上传大于8M的文件，只还得设置以下参数：
在Resource Limits 区域: max_execution_time = 600 ;每个PHP页面运行的最大时间值(秒)，默认30秒 max_input_time = 600 ;每个PHP页面接收数据所需的最大时间，默认60秒 memory_limit = 8m ;每个PHP页面所吃掉的最大内存，默认8M
