# Kasir
package com.danukris.kasir.Barang;


import android.app.ProgressDialog;
import android.content.Intent;
import android.support.design.widget.FloatingActionButton;
import android.support.v4.app.Fragment;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.AdapterView;
import android.widget.ListView;
import android.widget.Toast;

import com.android.volley.Request;
import com.android.volley.Response;
import com.android.volley.VolleyError;
import com.android.volley.toolbox.StringRequest;
import com.danukris.kasir.Config;
import com.danukris.kasir.MySingleton;
import com.danukris.kasir.R;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import java.util.ArrayList;

import static com.danukris.kasir.MainActivity.sqLiteHelper;

public class BarangActivity extends AppCompatActivity implements Response.Listener<String> {
    private static final String TAG = "BARANG";
    private AdapterView listviewDanu;
    ArrayList<DataModelBarang> listData = new ArrayList<DataModelBarang>();

    ProgressDialog pDialog;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_barang);
        pDialog     = new ProgressDialog(this);
        listviewDanu = (ListView) findViewById(R.id.listBarang);

        FloatingActionButton btnAddBarangDanu = (FloatingActionButton) findViewById(R.id.btnAddBarang);

        btnAddBarangDanu.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                addData();
            }
        });
    }
    @Override
    public void onResume() {
        super.onResume();
        fetchData();
    }
    private void fetchData(){
        pDialog.show();
        StringRequest stringRequest = new StringRequest(Request.Method.GET, Config.BASEURLAPI +"/TampilBarang.php", (Response.Listener<String>) this, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                pDialog.hide();
                Toast.makeText(getApplicationContext(), "Maaf terjadi kesalahan saat mengambil data, coba lagi.", Toast.LENGTH_SHORT).show();
            }
        });
        MySingleton.getInstance(this).addToRequestQueue(stringRequest);
    }

    private void addData(){
        Intent addData    = new Intent(this, AddBarangActivity.class);
        startActivity(addData);
    }
    private void previewData(int position){
        Intent preview   = new Intent(this, PriviewBarangActivity.class);
        DataModelBarang data   = (DataModelBarang) listData.get(position);
        preview.putExtra("data", data);
        startActivity(preview);
    }

    @Override
    public void onResponse(String response) {
        pDialog.hide();
        if (response != null) {
            try {
                JSONObject jsonObj  = new JSONObject(response);
                JSONObject items    = jsonObj.getJSONObject("items");
                JSONArray _datas = items.getJSONArray("data");

                int totalData = _datas.length();
                if(totalData > 0)
                {
                    listData.clear();
                    for (int i = 0; i < totalData; i++) {
                        JSONObject _data             = _datas.getJSONObject(i);
                        String _idBarang                = _data.getString("id");
                        String _kodeBarang              = _data.getString("kode_barang");
                        String _namaBarang              = _data.getString("nama");
                        String _satuanBarang            = _data.getString("satuan");
                        String _stokBarang              = _data.getString("stok");
                        String _harga_beliBarang        = _data.getString("harga_beli");
                        String _harga_jualBarang        = _data.getString("harga_jual");

                        int idBarang    =Integer.parseInt(_idBarang);
                        int stokBarang = Integer.parseInt(_stokBarang);
                        int harga_beliBarang = Integer.parseInt(_harga_beliBarang);
                        int harga_jualBarang = Integer.parseInt(_harga_jualBarang);

                        DataModelBarang dataMode =  new DataModelBarang(idBarang ,_kodeBarang, _namaBarang, _satuanBarang, stokBarang, harga_beliBarang, harga_jualBarang);
                        listData.add(dataMode);
                    }
                    ListAdapterBarang adapter = new ListAdapterBarang(this, listData);
                    listviewDanu.setAdapter(adapter);
                    listviewDanu.setOnItemClickListener(new AdapterView.OnItemClickListener() {
                        @Override
                        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                            previewData(position);
                        }
                    });
                }
            } catch (final JSONException e) {
                Log.e(TAG, "Error fetch data");
            }
        }
    }
}
