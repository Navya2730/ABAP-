# ABAP-
ABAP LEARNING
*****************************************************************************************
* Author      : Hemant Joshi  CG ID: CG21811
* Date        : 03.12.2025
* Reference   : ARDK940543
* Transport   : ARDK940543 / 13951
* FS No       : D2008
* Description : This is a utility program designed to find and repair certain errors in
*               master and financial data of ITSA taxpayer in ETMP, promoting data accuracy
*               and manual intervention .
*****************************************************************************************

REPORT ztr_itsa_health_check_utility MESSAGE-ID zetmp_health_check .

INCLUDE ztr_itsa_health_check_top .
INCLUDE ztr_itsa_health_check_sel_utl.
INCLUDE ztr_itsa_health_check_class.
INCLUDE ztr_itsa_health_report_class.
*-----------------------------------------------------------------------------------------*
*                                      AT SELECTION-SCREEN                                *
*-----------------------------------------------------------------------------------------*
AT SELECTION-SCREEN OUTPUT.
  lcl_health_check_execution=>selection_screen_pbo( )  .

*-----------------------------------------------------------------------------------------*
*                                      START OF SELECTION                                 *
*-----------------------------------------------------------------------------------------*
START-OF-SELECTION.
  TRY .

      DATA(gr_health_check) = NEW lcl_health_check_execution( it_regfb         = s_regfb[]    ##NEEDED
                                                              it_varfb         = s_varfb[]
                                                              it_retfb         = s_retfb[]
                                                              it_regevt        = s_regevt[]
                                                              it_varevt        = s_varevt[]
                                                              it_retevt        = s_retevt[]
                                                              iv_reg_chkbox    = p_reg
                                                              iv_ret_chkbox    = p_ret
                                                              iv_var_chkbox    = p_var
                                                              it_schedule_time = s_time[]
                                                              it_schedule_date = s_date[]
                                                              it_nino          = s_nino[] ) .
    CATCH zcx_itsa_health_check_utility INTO DATA(gr_exception) ##NEEDED .
      MESSAGE e050(zetmp_health_check) WITH gr_exception->get_text( ).
  ENDTRY.

  IF p_fnd IS NOT INITIAL. gr_health_check->scan_etmp_for_errors( ) . ENDIF .
  IF p_ref IS NOT INITIAL. gr_health_check->refresh_etmp( ) . ENDIF .


*-----------------------------------------------------------------------------------------*
*                                      END OF SELECTION                                   *
*-----------------------------------------------------------------------------------------*
END-OF-SELECTION.
  lcl_report=>display_errors( ).
* Save log
  zcl_health_log_errors=>get_instance( )->save( ).
