import os
import pandas as pd
from tkinter import Tk, filedialog, messagebox, ttk, filedialog
import tkinter as tk
import tkcalendar
import matplotlib.pyplot as plt
from datetime import datetime
import csv
from PIL import Image, ImageTk
import subprocess

class CSVCombinerApp:
    """
    CSV dict = {
    1 : value
    2 : value
    .
    .
    .
    n : value
    }
    """

    def __init__(self, master):
        self.master = master

        self.master.title("Coeus 0.2.8")
        self.master.iconbitmap("assets/logo.ico")
        self.master.geometry("600x600")
        self.master.tk.call('source', 'assets/forest-dark.tcl')
        
        style = ttk.Style(self.master)
        style.theme_use('forest-dark')


        self.main_page = tk.Frame(master)
        self.upload_page = tk.Frame(master)
        self.query_page = tk.Frame(master)
        self.plot_page = tk.Frame(master)
        self.r_page = tk.Frame(master)

        self.header_options = ['Hs', 'tau', 'u_star', 'Ts_stdev', 
                'Ts_Ux_cov', 'Ts_Uy_cov', 'Ts_Uz_cov', 'Ux_stdev', 'Ux_Uy_cov', 
                'Ux_Uz_cov', 'Uy_stdev', 'Uy_Uz_cov', 'Uz_stdev', 'wnd_spd', 
                'rslt_wnd_spd', 'wnd_dir_sonic', 'std_wnd_dir', 'wnd_dir_compass', 
                'Ux_Avg', 'Uy_Avg', 'Uz_Avg', 'Ts_Avg', 'sonic_azimuth', 
                'sonic_samples_Tot', 'no_sonic_head_Tot', 'no_new_sonic_data_Tot', 
                'amp_l_f_Tot', 'amp_h_f_Tot', 'sig_lck_f_Tot', 'del_T_f_Tot', 
                'aq_sig_f_Tot', 'sonic_cal_err_f_Tot', 'Fc_wpl', 'LE_wpl', 'Hc', 
                'CO2_stdev', 'CO2_Ux_cov', 'CO2_Uy_cov', 'CO2_Uz_cov', 'H2O_stdev', 
                'H2O_Ux_cov', 'H2O_Uy_cov', 'H2O_Uz_cov', 'Tc_stdev', 'Tc_Ux_cov', 
                'Tc_Uy_cov', 'Tc_Uz_cov', 'CO2_mean', 'H2O_mean', 'amb_tmpr_Avg', 
                'amb_press_mean', 'Tc_mean', 'rho_a_mean', 'Fc_irga', 'LE_irga', 
                'CO2_wpl_LE', 'CO2_wpl_H', 'H2O_wpl_LE', 'H2O_wpl_H', 
                'irga_samples_Tot', 'no_irga_head_Tot', 'no_new_irga_data_Tot', 
                'irga_bad_data_f_Tot', 'gen_sys_fault_f_Tot', 'sys_startup_f_Tot', 
                'motor_spd_f_Tot', 'tec_tmpr_f_Tot', 'src_pwr_f_Tot', 
                'src_tmpr_f_Tot', 'src_curr_f_Tot', 'irga_off_f_Tot', 
                'irga_sync_f_Tot', 'amb_tmpr_f_Tot', 'amb_press_f_Tot', 
                'CO2_I_f_Tot', 'CO2_Io_f_Tot', 'H2O_I_f_Tot', 'H2O_Io_f_Tot', 
                'CO2_Io_var_f_Tot', 'H2O_Io_var_f_Tot', 'CO2_sig_strgth_f_Tot', 
                'H2O_sig_strgth_f_Tot', 'CO2_sig_strgth_mean', 'H2O_sig_strgth_mean', 
                'T_tmpr_rh_mean', 'e_tmpr_rh_mean', 'e_sat_tmpr_rh_mean', 
                'H2O_tmpr_rh_mean', 'RH_tmpr_rh_mean', 'rho_a_tmpr_rh_mean', 
                'Rn_Avg', 'Rn_meas_Avg', 'Rain_TE525_Tot', 'LWS_mV_Avg', 
                'LWS_mV_Max', 'LWS_mV_Min', 'WetSeconds', 'WindSpd_WXT_mean', 
                'WindDir_WXT_mean', 'WindDir_WXT_StdDev', 'AirTemp_WXT_Avg', 
                'RH_WXT', 'AirPress_WXT', 'Rain_WXT_Tot', 'Hits_WXT_Tot', 
                'PAR_Den_Avg', 'PAR_Tot_Tot', 'panel_tmpr_Avg', 'batt_volt_Avg', 
                'slowsequence_Tot']

        
        """
        UPLOAD PAGE
        """

        self.upload_page_label = ttk.Label(self.upload_page, text="Upload Page", font=('Helvetica', 20)).pack(pady=5)

        self.label_dir = ttk.Label(self.upload_page, text="CSV Directory:")
        self.label_dir.pack(pady=5)
        
        self.entry_dir = ttk.Entry(self.upload_page, width=40)
        self.entry_dir.pack(pady=5)
        
        self.browse_button = ttk.Button(self.upload_page, text="Browse", command=self.browse_directory)
        self.browse_button.pack(pady=5)
        
        self.label_columns = ttk.Label(self.upload_page, text="Column names (comma-separated):")
        self.label_columns.pack(pady=5)
        
        self.entry_columns = ttk.Entry(self.upload_page, width=40)
        self.entry_columns.pack(pady=5)
        
        self.combine_button = ttk.Button(self.upload_page, text="Combine CSVs", command=self.combine_csv)
        self.combine_button.pack(pady=20)

        ttk.Button(self.upload_page, text='Go to Main Page', command=lambda:self.show_page(self.main_page)).pack(pady=5)

        """
        MAIN PAGE
        """

        self.main_page_label = ttk.Label(self.main_page, text="ULM Plant Ecology Lab", font=('Serif', 20)).pack(pady=5)

        self.image = Image.open("assets/ludwigiasphinx.jpg")
        #self.image = self.image.resize((200, 150), Image.ANTIALIAS)
        self.image = ImageTk.PhotoImage(self.image)

        self.image_label = ttk.Label(self.main_page, image=self.image)
        self.image_label.pack(pady=5)
        
        self.upload_page_button = ttk.Button(self.main_page, text='Go to Upload Page',style='Accent.TButton', command=lambda:self.show_page(self.upload_page)).pack(pady=5)
        self.query_page_button = ttk.Button(self.main_page, text='Go to Query Page', command=lambda:self.show_page(self.query_page)).pack(pady=5)
        self.plot_page_button = ttk.Button(self.main_page, text='Go to Plot Page', command=lambda:self.show_page(self.plot_page)).pack(pady=5)
        self.r_page_button = ttk.Button(self.main_page, text='Go to R Page', command=lambda:self.show_page(self.r_page)).pack(pady=5)

        """
        QUERY PAGE
        """
        self.query_page_label = ttk.Label(self.query_page, text='Query Page').pack(pady=5)

        self.label_year = ttk.Label(self.query_page, text="Query Year")
        self.label_year.pack(pady=5)

        self.label_from = ttk.Label(self.query_page, text="From")
        self.label_from.pack(pady=5)
        
        self.entry_date_from = tkcalendar.DateEntry(self.query_page)
        self.entry_date_from.pack(pady=5)

        self.label_to = ttk.Label(self.query_page, text="To")
        self.label_to.pack(pady=5)

        self.entry_date_to = tkcalendar.DateEntry(self.query_page)
        self.entry_date_to.pack(pady=5)

        ttk.Button(self.query_page, text='Select All', command=lambda:self.select_all()).pack(pady=5)

        self.listbox = tk.Listbox(self.query_page, selectmode=tk.MULTIPLE)
        self.listbox.pack(pady=5)

        for item in self.header_options:
            self.listbox.insert(tk.END, item)

        self.listbox.bind("<<ListboxSelect>>", self.on_select)
        

        self.query_button= ttk.Button(self.query_page, text='Query', command=lambda:self.find_data_at_date(self.entry_date_from.get(), self.entry_date_to.get())).pack(pady=5)
        
        #TODO
        ttk.Button(self.query_page, text='Eddy Out', command=lambda:self.eddy_export()).pack(pady=5)

        ttk.Button(self.query_page, text='Go to Main Page', command=lambda:self.show_page(self.main_page)).pack(pady=5)



        """
        PLOT PAGE
        """


        ttk.Label(self.plot_page, text="Plot Page").pack(pady=5)

        # can make a function that lists all pages
        # csv_options = get_csv_list()
        csv_options = ['combined_data.csv', 'query/queried_data.csv']

        selected_csv = tk.StringVar(value=csv_options[0])

        csv_dropdown = tk.OptionMenu(self.plot_page, selected_csv, *csv_options)
        csv_dropdown.pack(pady=5)

        # can make a function that lists all headers
        # header_options = get_header_list

        plot_options = ['date'] + self.header_options
        
        ttk.Label(self.plot_page, text='x_value:').pack(pady=5)
        selected_x = tk.StringVar(value=plot_options[0])
        x_dropdown = tk.OptionMenu(self.plot_page, selected_x, *plot_options)
        x_dropdown.pack(pady=5)

        ttk.Label(self.plot_page, text='y_value:').pack(pady=5)
        selected_y = tk.StringVar(value=plot_options[0])
        y_dropdown = tk.OptionMenu(self.plot_page, selected_y, *plot_options)
        y_dropdown.pack(pady=5)

        type_options = ['line', 'bar', 'barh', 'hist', 'box', 'kde', 'density', 'area', 'pie', 'scatter', 'hexbin']

        ttk.Label(self.plot_page, text='Type of graph').pack(pady=5)
        selected_type = tk.StringVar(value=type_options[0])
        type_dropdown = tk.OptionMenu(self.plot_page, selected_type, *type_options)
        type_dropdown.pack(pady=5)

        ttk.Button(self.plot_page, text='Get Plot', command=lambda:self.plot_data(selected_csv.get(),selected_x.get(),selected_y.get(), selected_type.get())).pack(pady=5)


        ttk.Button(self.plot_page, text='Go to Main Page', command=lambda:self.show_page(self.main_page)).pack(pady=5)


        """
        R Functionality
        """
        ttk.Label(self.r_page, text="R Page").pack(pady=5)
        csv_dropdown = tk.OptionMenu(self.r_page, selected_csv, *csv_options)
        csv_dropdown.pack(pady=5)
        self.label_r_file = ttk.Label(self.r_page, text="Select R file:")
        self.label_r_file.pack(pady=5)
        self.entry_r_file = ttk.Entry(self.r_page, width=40)
        self.entry_r_file.pack(pady=5)
        self.browse_r_button = ttk.Button(self.r_page, text="Browse R File", command=self.browse_r_file)
        self.browse_r_button.pack(pady=5)

        ttk.Button(self.r_page, text='Execute R Script', command=lambda:self.execute_r_script(self.entry_r_file.get(), selected_csv.get())).pack(pady=5)


        ttk.Button(self.r_page, text='Go to Main Page', command=lambda:self.show_page(self.main_page)).pack(pady=5)



        
        self.show_page(self.main_page)


    ################################
    ########  BACKEND LOGIC ########
    ################################
    
    def execute_r_script(self, r_script_path, csv_file_path):
        try:
            result = subprocess.run(
                ["bash", "run_r_script.sh", r_script_path, csv_file_path],
                check=True,
                capture_output=True,
                text=True
            )
            print("R script output:", result.stdout)
        except subprocess.CalledProcessError as e:
            print("Error executing R script:", e.stderr)
    
    # Example usage
    # self.execute_r_script("path/to/your/script.R", "path/to/your/data.csv")

    def eddy_export(self):

        #TODO wtf is P_RAIN FIX THE DAMN DATAPOINTS. STAY ON TRACK WTIH THE MOVING OF COLUMNS 
        #THEN CONTINUE OUTPUTTING THE MOVED AROUND DATA TO A CSV :D
        csv_file = 'query/queried_data.csv'
        df = pd.read_csv(csv_file)

        #_df = pd.DataFrame({'TIMESTAMP_1','TIMESTAMP_2','DOY','Rn','LWS','Ta','RH','Pa','P_rain','PPFD','SWin'})
        #_df2 = pd.DataFrame({'yyyy-mm-dd','HHMM','ddd','W+1m-2','mV','C','%','kPa','mm','umol+1m-2s-1','W+1m-2'})
        header_row = ['TIMESTAMP_1','TIMESTAMP_2','DOY','Rn','LWS','Ta','RH','Pa','P_rain']
        unit_row = ['yyyy-mm-dd','HHMM','ddd','W+1m-2','mV','C','%','kPa','mm']

        times = df['time']
        times_list = []
        for time in times:
            times_list.append(str(time).zfill(4))

        
        df['time'] = times_list
        
        dates = []
        for i in range(len(df['year'])):
            dates.append(datetime.strptime(str(df['year'][i])+'/'+str(df['day'][i]), '%Y/%j').date())
        df['year'] = dates
        df = df[['year', 'time', 'day', 'Rn_meas_Avg', 'LWS_mV_Avg', 'Tc_mean', 'RH_tmpr_rh_mean']]
        #new_row = ['yyyy-mm-dd','HHMM','ddd','W+1m-2','mV','C','%','kPa','mm','umol+1m-2s-1','W+1m-2']
        #df = pd.concat([df.iloc[:1], pd.DataFrame([new_row]), df.iloc[1:]], ignore_index=False)
        
        #df.columns = ['TIMESTAMP_1','TIMESTAMP_2','DOY','Rn','LWS','Ta','RH','Pa','P_rain']
        df.columns = ['TIMESTAMP_1','TIMESTAMP_2','DOY','Rn','LWS','Ta','RH']
        with open('sample_eddy.csv','w', newline='') as f:
            writer = csv.writer(f)
            writer.writerow(header_row)
            writer.writerow(unit_row)
        
        _df = pd.read_csv('sample_eddy.csv')
        result = pd.concat([_df, df], ignore_index=True)
        result.to_csv('eddy.csv')
            #for index, row in df.iterrows():
            #    if index==0:
            #        continue
            #    writer.writerow()


    def plot_data(self, csv_path, x_value, y_value, type_value):
        
        df = pd.read_csv(str(csv_path))

        # TODO SCALE THIS TO NOT JUST DO X_VALUE AND BE MORE INTUITIVE FOR USER INPUT
        # CAN BE PUT IN QUERY FUNCTIONALITY BUT WITH BUTTON PROMPT SO IT ISNT DEFAULT
        dates = []
        if x_value == 'date':
            for i in range(len(df['day'])):
                dates.append(self.normalized_date_time_string(df['year'][i], df['day'][i], df['time'][i]))
            
            df.insert(0, 'date', dates)
            df = df[(df[y_value] > -1000)]

        else:
            df = df[(df[x_value] > -1000) & (df[y_value] > -1000)]

        df.plot(x=x_value, y=y_value, kind=type_value)
        plt.show()

    def normalized_date_time_string(self, year, day, time):
        time = str(time).zfill(4)
        time = (time)[:2] + ':' + (time)[2:]
        return datetime.strptime(str(year) + '/' + str(day) + ' ' + time,'%Y/%j %H:%M')


    def day_of_year(self, date_str):
        date_obj = datetime.strptime(date_str, "%m/%d/%y")
        return date_obj.timetuple().tm_yday, date_obj.year

    def find_data_at_date(self, date_from, date_to):

        day_from, year_from = self.day_of_year(str(date_from))
        day_to, year_to = self.day_of_year(str(date_to))

        csv_path = 'combined_data.csv'
        _df = pd.read_csv(csv_path)

        df = _df[self.get_selected_items()]
        df.insert(0, 'year', _df['year'])
        df.insert(1, 'day', _df['day'])
        df.insert(2, 'time', _df['time'])

        dfs = []
        
        for i in range(int(year_from),int(year_to)+1):
            dfs.append(df.loc[(df['year'] == i) & (df['day'] >= day_from) & (df['day'] <= day_to)])  
        
        result = pd.concat(dfs ,ignore_index=True,axis=0)
        
        result = result[['year', 'day', 'time'] + self.get_selected_items()]
        result.to_csv('query/queried_data.csv', index=False)

    
    ###############################
    #########  GUI LOGIC ##########
    ###############################

    def select_all(self):
        self.listbox.selection_set(0, tk.END)

    def get_selected_items(self):
        selected_indices = self.listbox.curselection()
        selected_values = [self.listbox.get(index) for index in selected_indices]
        return selected_values

    def on_select(self, event):
        selected_items = self.get_selected_items()



    def get_header_list(self):
        pass

    """
    def search_files(self):
        csv_files = [f for f in os.listdir(directory) if f.endswith(".csv")]
        result = []

        os.getcwd()

        for root, dirs, files in os.walk(os.getcwd):
            if files.endswith('csv'):
                result.append(os.path.join(, file_name))
        return result
    """

        


    def show_page(self, page):
        # hide all pages
        self.main_page.pack_forget()
        self.upload_page.pack_forget()
        self.query_page.pack_forget()
        self.plot_page.pack_forget()
        self.r_page.pack_forget()

        # show specified page
        page.pack()

    def browse_r_file(self):
        file_path = filedialog.askopenfilename(
            title="Select R File",
            filetypes=(("R files", "*.R"), ("All files", "*.*"))
        )
        if file_path:
            self.entry_r_file.delete(0, "end")
            self.entry_r_file.insert(0, file_path)
        


    def browse_directory(self):
        directory = filedialog.askdirectory()
        if directory:
            self.entry_dir.delete(0, "end")
            self.entry_dir.insert(0, directory)
    
    def combine_csv(self):
        directory = self.entry_dir.get()

        column_names = self.entry_columns.get().split(",")
        
        if not os.path.isdir(directory):
            messagebox.showerror("Error", "Invalid directory!")
            return
        
        csv_files = [f for f in os.listdir(directory) if f.endswith(".csv")]

        for i in range(len(csv_files)):
            csv_files[i] = os.path.join(directory, csv_files[i])

        print(csv_files)

        if not csv_files:
            messagebox.showerror("Error", "No CSV files found in the directory!")
            return
        
        dfs = []

        # Loop over each CSV file and read it into a dataframe
        for file in csv_files:

            df = pd.read_csv(file)

            df = pd.read_csv(file, names=['year', 'day', 'time', 'Hs', 'tau', 'u_star', 'Ts_stdev', 
                'Ts_Ux_cov', 'Ts_Uy_cov', 'Ts_Uz_cov', 'Ux_stdev', 'Ux_Uy_cov', 
                'Ux_Uz_cov', 'Uy_stdev', 'Uy_Uz_cov', 'Uz_stdev', 'wnd_spd', 
                'rslt_wnd_spd', 'wnd_dir_sonic', 'std_wnd_dir', 'wnd_dir_compass', 
                'Ux_Avg', 'Uy_Avg', 'Uz_Avg', 'Ts_Avg', 'sonic_azimuth', 
                'sonic_samples_Tot', 'no_sonic_head_Tot', 'no_new_sonic_data_Tot', 
                'amp_l_f_Tot', 'amp_h_f_Tot', 'sig_lck_f_Tot', 'del_T_f_Tot', 
                'aq_sig_f_Tot', 'sonic_cal_err_f_Tot', 'Fc_wpl', 'LE_wpl', 'Hc', 
                'CO2_stdev', 'CO2_Ux_cov', 'CO2_Uy_cov', 'CO2_Uz_cov', 'H2O_stdev', 
                'H2O_Ux_cov', 'H2O_Uy_cov', 'H2O_Uz_cov', 'Tc_stdev', 'Tc_Ux_cov', 
                'Tc_Uy_cov', 'Tc_Uz_cov', 'CO2_mean', 'H2O_mean', 'amb_tmpr_Avg', 
                'amb_press_mean', 'Tc_mean', 'rho_a_mean', 'Fc_irga', 'LE_irga', 
                'CO2_wpl_LE', 'CO2_wpl_H', 'H2O_wpl_LE', 'H2O_wpl_H', 
                'irga_samples_Tot', 'no_irga_head_Tot', 'no_new_irga_data_Tot', 
                'irga_bad_data_f_Tot', 'gen_sys_fault_f_Tot', 'sys_startup_f_Tot', 
                'motor_spd_f_Tot', 'tec_tmpr_f_Tot', 'src_pwr_f_Tot', 
                'src_tmpr_f_Tot', 'src_curr_f_Tot', 'irga_off_f_Tot', 
                'irga_sync_f_Tot', 'amb_tmpr_f_Tot', 'amb_press_f_Tot', 
                'CO2_I_f_Tot', 'CO2_Io_f_Tot', 'H2O_I_f_Tot', 'H2O_Io_f_Tot', 
                'CO2_Io_var_f_Tot', 'H2O_Io_var_f_Tot', 'CO2_sig_strgth_f_Tot', 
                'H2O_sig_strgth_f_Tot', 'CO2_sig_strgth_mean', 'H2O_sig_strgth_mean', 
                'T_tmpr_rh_mean', 'e_tmpr_rh_mean', 'e_sat_tmpr_rh_mean', 
                'H2O_tmpr_rh_mean', 'RH_tmpr_rh_mean', 'rho_a_tmpr_rh_mean', 
                'Rn_Avg', 'Rn_meas_Avg', 'Rain_TE525_Tot', 'LWS_mV_Avg', 
                'LWS_mV_Max', 'LWS_mV_Min', 'WetSeconds', 'WindSpd_WXT_mean', 
                'WindDir_WXT_mean', 'WindDir_WXT_StdDev', 'AirTemp_WXT_Avg', 
                'RH_WXT', 'AirPress_WXT', 'Rain_WXT_Tot', 'Hits_WXT_Tot', 
                'PAR_Den_Avg', 'PAR_Tot_Tot', 'panel_tmpr_Avg', 'batt_volt_Avg', 
                'slowsequence_Tot'])
            
            dfs.append(df)

        # Concatenate all dataframes into a single one
        combined_df = pd.concat(dfs ,ignore_index=True,axis=0)
        # Save the combined dataframe to a new CSV file
        combined_df.to_csv('combined_data.csv', index=False)

if __name__ == "__main__":
    root = Tk()
    app = CSVCombinerApp(root)
    root.mainloop()
