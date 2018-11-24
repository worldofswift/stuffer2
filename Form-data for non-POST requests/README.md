# Non-POST
По-умолчанию, если тело Вашего запроса имеет тип form-data и это не GET/POST запросы, то оно не будет доступно на сервере.
Но следующий код решает проблему. 

> Оверхед для обычных запросов - один вызов функции и 2 сравнения, 1 логическая операция

## 1. Создайте класс для парсинга данных 
```php
<?php /** @noinspection PhpUndefinedClassInspection */

namespace App\Extensions;

use Illuminate\Support\Facades\Log;
use Symfony\Component\HttpFoundation\File\UploadedFile;

/**
 * Class ParseInputStream
 *
 * Intended to be used for solve problem with php patch data.
 *
 * @package App\Services
 */
class ParseInputStream
{
    /**
     * @abstract Raw input stream
     */
    protected $input;

    /**
     * @function __construct
     *
     * @param array $data stream
     */
    public function __construct(array &$data)
    {
        $this->input = file_get_contents('php://input');
        $boundary = $this->boundary();
        if ('' === $boundary) {
            $data = [
                'parameters' => $this->parse(),
                'files' => []
            ];
        } else {
            $blocks = $this->split($boundary);
            $data = $this->blocks($blocks);
        }
    }
    /**
     * @function boundary
     * @return string
     */
    private function boundary(): string
    {
        if(!isset($_SERVER['CONTENT_TYPE'])) {
            return null;
        }
        preg_match('/boundary=(.*)$/', $_SERVER['CONTENT_TYPE'], $matches);
        return $matches[1];
    }
    /**
     * @function parse
     * @returns array
     */
    private function parse()
    {
        parse_str(urldecode($this->input), $result);
        return $result;
    }

    /**
     * @function split
     * @param string $boundary
     * @return array[]|false|string[]
     */
    private function split(string $boundary)
    {
        $result = preg_split("/-+$boundary/", $this->input);
        array_pop($result);
        return $result;
    }

    /**
     * @function blocks
     * @param array $array
     * @return array
     */
    private function blocks(array $array): array
    {
        $results = [];
        foreach($array as $value) {
            if (empty($value)) {
                continue;
            }
            $block = $this->decide($value);
            foreach ($block['parameters'] as $key => $val) {
                $this->parse_parameter( $results, $key, $val );
            }
            foreach ($block['files'] as $key => $val) {
                $this->parse_parameter( $results, $key, $val );
            }
        }
        return $results;
    }

    /**
     * @function decide
     * @param string $string
     * @return array
     */
    private function decide(string $string): array
    {
        if (strpos($string, 'application/octet-stream') !== false)
        {
            return [
                'parameters' => $this->file($string),
                'files' => []
            ];
        }
        if (strpos($string, 'filename') !== FALSE)
        {
            return [
                'parameters' => [],
                'files' => $this->file_stream($string)
            ];
        }
        return [
            'parameters' => $this->parameter($string),
            'files' => []
        ];
    }
    /**
     * @function file
     *
     * @param $string
     *
     * @return array
     */
    private function file($string): array
    {
        preg_match('/name=\"([^\"]*)\".*stream[\n|\r]+([^\n\r].*)?$/s', $string, $match);
        return [
            $match[1] => $match[2] ?? ''
        ];
    }

    /**
     * @function file_stream
     *
     * @param $data
     *
     * @return array
     */
    private function file_stream($data): array
    {
        $result = [];
        $data = ltrim($data);
        $idx = strpos( $data, "\r\n\r\n" );
        if ($idx) {
            $headers = substr( $data, 0, $idx );
            $content = substr( $data, $idx + 4, -2 ); // Skip the leading \r\n and strip the final \r\n
            $name = '-unknown-';
            $filename = '-unknown-';
            $filetype = 'application/octet-stream';
            $header = strtok( $headers, "\r\n" );
            while ($header !== false) {
                if (strncmp($header, 'Content-Disposition: ', 21) === 0) {
                    // Content-Disposition: form-data; name="attach_file[TESTING]"; filename="label2.jpg"
                    if ( preg_match('/name=\"([^\"]*)\"/', $header, $nmatch ) ) {
                        $name = $nmatch[1];
                    }
                    if ( preg_match('/filename=\"([^\"]*)\"/', $header, $nmatch ) ) {
                        $filename = $nmatch[1];
                    }
                } elseif (strncmp($header, 'Content-Type: ', 14) === 0) {
                    // Content-Type: image/jpg
                    $filetype = trim( substr($header, \strlen('Content-Type: ')) );
                } else {
                    Log::debug( 'PARSEINPUTSTREAM: Skipping Header: ' . $header );
                }
                $header = strtok("\r\n");
            }
            if (substr($data, -2) === "\r\n") {
                /** @noinspection PhpUnusedLocalVariableInspection */
                $data = substr($data, 0, -2);
            }
            $path = sys_get_temp_dir() . '/php' . substr( sha1(mt_rand()), 0, 6 );
            $bytes = file_put_contents( $path, $content );
            if ( $bytes !== false ) {
                $file = new UploadedFile( $path, $filename, $filetype, $bytes, UPLOAD_ERR_OK );
                $result = [$name => $file];
            }
        } else {
            Log::warning('ParseInputStream.file_stream(): Could not locate header separator in data:');
            Log::warning( $data );
        }
        return $result;
    }
    /**
     * @function parameter
     *
     * @param $string
     *
     * @return array
     */
    private function parameter($string): array
    {
        $data = [];
        if ( preg_match('/name=\"([^\"]*)\"[\n|\r]+([^\n\r].*)?\r$/s', $string, $match) ) {
            if (preg_match('/^(.*)\[\]$/', $match[1], $tmp)) {
                $data[$tmp[1]][] = ($match[2] ?? '');
            } else {
                $data[$match[1]] = ($match[2] ?? '');
            }
        }
        return $data;
    }

    /**
     * @param $params
     * @param $parameter
     * @param $value
     */
    public function parse_parameter(&$params, $parameter, $value ): void
    {
        if (strpos($parameter, '[') !== false) {
//            $matches = array();
            if ( preg_match( '/^([^[]*)\[([^]]*)\](.*)$/', $parameter, $match ) ) {
                [$name, $key, $rem] = $match;
                if ( $name !== '' && $name !== null ) {
                    if (!isset($params[$name]) || ! \is_array($params[$name])) {
                        $params[$name] = [];
                    }

                    if ($rem !== '') {
                        if ($key === '' || $key === null) {
                            $arr = [];
                            $this->parse_parameter( $arr, $rem, $value );
                            $params[$name][] = $arr;
                        } else {
                            if ( !isset($params[$name][$key]) || !\is_array($params[$name][$key]) ) {
                                $params[$name][$key] = [];
                            }
                            $this->parse_parameter( $params[$name][$key], $rem, $value );
                        }
                    } else if ($key === '' || $key === null) {
                        $params[$name][] = $value;
                    } else {
                        $params[$name][$key] = $value;
                    }
                } else if ($rem !== '') {
                    if ($key === '' || $key === null) {
                        // REVIEW Is this logic correct?!
                        $this->parse_parameter( $params, $rem, $value );
                    } else {
                        if ( ! isset($params[$key]) || ! \is_array($params[$key]) ) {
                            $params[$key] = [];
                        }
                        $this->parse_parameter( $params[$key], $rem, $value );
                    }
                } else if ( $key === '' || $key === null ) {
                    $params[] = $value;
                } else {
                    $params[$key] = $value;
                }
            } else {
                Log::warning( "ParseInputStream.parse_parameter() Parameter name regex failed: '" . $parameter . "'" );
            }
        } else {
            $params[$parameter] = $value;
        }
    }
}

```

## 2. Создайте чудесный Middleware
```php
<?php

namespace App\Http\Middleware;

use App\Extensions\ParseInputStream;
use Closure;

/**
 * Class ParseMultipartFormDataInputForNonPostRequests
 * @package App\Http\Middleware
 */
class ParseMultipartFormDataInputForNonPostRequests
{
    /**
     * @param $request
     * @param Closure $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        /** @noinspection PhpUndefinedMethodInspection */
        $requestMethod = $request->method();
        if ($requestMethod === 'POST' || $requestMethod === 'GET') {
            return $next($request);
        }

        if (preg_match('/multipart\/form-data/', $request->headers->get('Content-Type')) ||
            preg_match('/multipart\/form-data/', $request->headers->get('content-type'))
        ) {
            $params = [];
            new ParseInputStream($params);
            $request->request->add($params);

        }
        return $next($request);
    }
}

```

## [3]. Добавьте его ко всем запросам, если есть такая необходимость > /app/Http/Kernel.php > middleware
```php
ParseMultipartFormDataInputForNonPostRequests::class,
``` 
